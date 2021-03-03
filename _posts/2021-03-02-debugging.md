---
layout: post
title:  "Coverage debugging"
date:   2021-03-02 10:00:28 -0500
categories: jekyll update
---

### Case 1 ###

In the below code snippet in ```binutils```. Branch at Line 309 is a concrete branch, so no constraint is generated for the branch. However, the branch can be flipped by flipping Line 509 in ```bfd/archive.c```, which tries to match ```hdr.ar_fmag``` to ```ARFMAG```. ```ARFMAG``` is a string literal, defined as ```#define ARFMAG "`\012"```.  Note that ```\012``` is a octal escape sequence, which is "\n" according to ascii table. 

```
 -:  301:bfd *
      210:  302:_bfd_look_for_bfd_in_cache (bfd *arch_bfd, file_ptr filepos)
        -:  303:{
      210:  304:  htab_t hash_table = bfd_ardata (arch_bfd)->cache;
        -:  305:  struct ar_cache m;
        -:  306:
      210:  307:  m.ptr = filepos;
        -:  308:
      210:  309:  if (hash_table)
        -:  310:    {
        3:  311:      struct ar_cache *entry = (struct ar_cache *) htab_find (hash_table, &m);
        3:  312:      if (!entry)
        3:  313:        return NULL;
        -:  314:
        -:  315:      /* Unfortunately this flag is set after checking that we have
        -:  316:         an archive, and checking for an archive means one element has
        -:  317:         sneaked into the cache.  */
    #####:  318:      entry->arbfd->no_export = arch_bfd->no_export;
    #####:  319:      return entry->arbfd;
        -:  320:    }
        -:  321:  else
      207:  322:    return NULL;
        -:  323:}
```

```
 636:  505:  if (strncmp (hdr.ar_fmag, ARFMAG, 2) != 0
      633:  506:      && (mag == NULL
    #####:  507:          || strncmp (hdr.ar_fmag, mag, 2) != 0))
        -:  508:    {
      633:  509:      bfd_set_error (bfd_error_malformed_archive);
      633:  510:      return NULL;
        -:  511:    }
 ```
 
 **The first questions is, what is the relationship between target branch of the related branch?**

Set a breakpoint in function ```_bfd_look_for_bfd_in_cache```, print out the content

```
(gdb) p *arch_bfd.tdata.aout_ar_data
$14 = {first_file_filepos = 8, cache = 0x5555558dfbc0, archive_head = 0x0, symdefs = 0x0, symdef_count = 0, extended_names = 0x0, extended_names_size = 0, armap_timestamp = 0, armap_datepos = 0, tdata = 0x0}
```
Set a break point on ```cache```, and found that it is updated at ```archive.c:358```

```
363           hash_table = htab_create_alloc (16, hash_file_ptr, eq_file_ptr,
```

The check the callback

```
(gdb) bt
#0  _bfd_add_bfd_to_archive_cache (arch_bfd=0x5555558da630, filepos=8, new_elt=0x5555558dda80) at archive.c:363
#1  0x000055555558859e in _bfd_get_elt_at_filepos (archive=0x5555558da630, filepos=8) at archive.c:749
#2  0x0000555555588742 in bfd_generic_openr_next_archived_file (archive=0x5555558da630, last_file=0x0) at archive.c:837
#3  0x000055555558869d in bfd_openr_next_archived_file (archive=0x5555558da630, last_file=0x0) at archive.c:805
#4  0x0000555555585237 in display_archive (file=0x5555558da630) at size.c:383
#5  0x0000555555585339 in display_file (filename=0x7fffffffe4bc "/home/cju/test/id-00000056") at size.c:432
#6  0x0000555555584de6 in main (argc=2, argv=0x7fffffffe148) at size.c:260
```

Then tracked ```_bfd_get_elt_at_filepos```, found that the flow exits at this check

```
if ((new_areldata = (struct areltdata *) _bfd_read_ar_hdr (archive)) == NULL)
  return NULL;
```

```bfd_read_ar_hdr``` wraps a function pointer, the function get called is actually ```_bfd_generic_read_ar_hdr_mag```.  Inside this function, it is exactly that the check at Line 509 fails, so that it returns a ```NULL```

**At this point, I arrive the conclusion for the first question, that the target branch has control flow dependency over check on archive:509**

**The second question is, if the target branch can be flipped? Why is it not flipped in the fuzzing?**

I first check is there any seed the fuzzing output directory which has byte 66/67 as '`' and `\n`, basically after solving the archive:509 branch. I get the following python script to scan the directory

```
import os
import glob
for filename in glob.glob('id*'):
   with open(os.path.join(os.getcwd(), filename), 'rb') as f: # open in readonly mode
      f.seek(66)
      a=f.read(2)
      if len(a) == 2 and a[0] == '`' and a[1] == '\n':
         print ("found"+filename)
```

Suprising! There is no such file which containts these two bytes. Why?

Before long I am reminded that the magic bytes could be at other byte offsets, ```hdr.ar_fmag``` might be pointing to other places in the input.

I then use the tool bgrep (https://github.com/tmbinc/bgrep) to grep the magic bytes in all inputs (for some reason grep does not work on the binary string \x60\x0a ), and indeed, there are inputs that contain these magic bytes, but none of these bytes located at position 66,67 (0x42,0x43).

```
cju@kili:~/e2e_data/jigsaw_size16/angora/queue$ ./bgrep 600a *
id:002639: 00000e0a
id:006707: 00000e0a
id:006708: 00000e0e
id:006709: 00000e0a
id:008634: 00000e0a
id:009578: 00000611
id:010906: 00000e0a
id:011130: 00000e0a
id:012462: 00000e0a
id:012478: 00000e0a
id:016147: 00000e0a
```

But Angora does have such inputs

```
cju@kili:~/e2e_data/ang_size1/queue$ ../../jigsaw_size16/angora/queue/bgrep 600a *
id:006811: 00000042
id:010531: 00000082
id:010532: 00000042
id:010533: 00000042
id:010533: 00000082
id:010534: 00000042
id:010554: 00000042
id:010554: 00000082
id:010557: 00000042
id:010557: 00000082
id:010558: 00000042
id:010558: 00000082
id:010566: 00000042
id:010567: 00000042
id:010575: 00000042
id:010576: 00000042
id:010577: 00000042
id:010578: 00000042
id:010646: 00000042
id:017022: 00000042
id:017022: 0000007e
id:021229: 00001c18
```
