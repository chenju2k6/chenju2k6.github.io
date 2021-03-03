---
layout: post
title:  "Coverage debugging"
date:   2021-03-02 10:00:28 -0500
categories: jekyll update
---

### Coverage Debugging ###

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
 
 **The first questions is, what is the relationship between target branch of the related branch?***

Set a breakpoint in function ```_bfd_look_for_bfd_in_cache```, print out the content

```
(gdb) p *arch_bfd.tdata.aout_ar_data
$14 = {first_file_filepos = 8, cache = 0x5555558dfbc0, archive_head = 0x0, symdefs = 0x0, symdef_count = 0, extended_names = 0x0, extended_names_size = 0, armap_timestamp = 0, armap_datepos = 0, tdata = 0x0}
```
Set a break point on ```cache```, and found that it is updated at ```archive.c:358```

```
363           hash_table = htab_create_alloc (16, hash_file_ptr, eq_file_ptr,
```

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

**At this point, I arrive the conclusion for the first question, that the target branch has control flow dependency over check on archive:509*** 
