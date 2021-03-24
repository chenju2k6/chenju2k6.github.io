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

**At this point, I make a guess that the branch solving request for this magic number matching at byte offset [66,67] is filtered out because an earlier flipping request is already handled for the same branch but it tries to match magic number at different byte locations**

But is the guess right? I then have a script to fetch all the seeds in the fuzzing output directory which can reach branch archive.c:504. What I do is to insert a line ```exit(55);``` before the branch archive.c:504 and then check the exit status after executing the seed.

```
dir=$1
find ${dir} -maxdepth 1  -name "id\:*" -printf "%T@ %Tc %p\n"  | sort -n  | cut -d " " -f 9 | while read line; do
    ./size  $line 1>/dev/null 2>/dev/null
    if [ $? -eq 55 ]
    then
    echo "find seed arrives at target branch " $line
fi
done
```

The output is 

```
find seed arrives at target branch corpus/angora/queue/id:000323
find seed arrives at target branch corpus/angora/queue/id:000324
find seed arrives at target branch corpus/angora/queue/id:000939
find seed arrives at target branch corpus/angora/queue/id:000940
find seed arrives at target branch corpus/angora/queue/id:000941
find seed arrives at target branch corpus/angora/queue/id:000942
```

The first seed in the output dir which can reach the branch is id:000323. If I am guessing right, the branch constraints for archive.c:505 for this seed involves different bytes other than [66,67]

I then flip all the branches for id:000323. And use ```bgrep``` to grep the magic number for all the inputs, and then I found this

```
id-00000056: 00000042
```

**Which says that the bytes mutated is indeed for bytes [66,67], suggesting my guess is WRONG!, so what is happening?**

First, I get the address for the branch by looking the log

```
generate fmemcmp input 56 addr 0x7000002479db
```

Then, I added more print outs in the fuzzing log. **Looking the log, I found that the solving request is indeeded solved for input id:000323, but the resulting test case is not saved by the grader, why?** 
```
***INFO  fastgen::fuzz_loop    > grading input derived from on input 323 by flipping branch@ 0x7000002479db ctx 0x0 order 0, it is a new input false, saved as input #0***
 INFO  fastgen::fuzz_loop    > grading input derived from on input 325 by flipping branch@ 0x700000660def ctx 0x56be1bc4 order 1, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 324 by flipping branch@ 0x70000024e421 ctx 0x0 order 0, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 6, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 6, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 7, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 7, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003df14d ctx 0x210bf9f9 order 7, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003df14d ctx 0x210bf9f9 order 7, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 8, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 8, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003df14d ctx 0x210bf9f9 order 8, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003df14d ctx 0x210bf9f9 order 8, it is a new input false, saved as input #0
 INFO  fastgen::fuzz_loop    > grading input derived from on input 329 by flipping branch@ 0x7000003bbc82 ctx 0x6b01ab4 order 9, it is a new input true, saved as input #954
 ```

### Case 2 ###

In libxml2, Angora covers the true direction of the below branch but we can't.

```
if (cur->oldNs != NULL)  { xmlFreeNsList(cur->oldNs); }
```

Fetch the seed which can cover the true direction of this branch, run the seed, found that this is a concrete branch. 

Setting a watch point at the address of cur->oldNs, found that it is updated at the function of ```xmlTreeEnsureXMLDecl```

```doc->oldNs = ns;```

The callstack is 

```
(gdb) bt
#0  xmlTreeEnsureXMLDecl (doc=<optimized out>) at tree.c:5960
#1  0x00005555555fa996 in xmlSAX2StartElementNs (ctx=0x55555588caa0, localname=<optimized out>, prefix=<optimized out>,
    URI=<optimized out>, nb_namespaces=<optimized out>, namespaces=<optimized out>, nb_attributes=0, nb_defaulted=<optimized out>,
    attributes=<optimized out>) at SAX2.c:2359
#2  0x000055555557bddb in xmlParseStartTag2 (ctxt=ctxt@entry=0x55555588caa0, pref=pref@entry=0x7fffffffdf20,
    URI=URI@entry=0x7fffffffdf28, tlen=tlen@entry=0x7fffffffdf1c) at parser.c:9707
#3  0x0000555555580610 in xmlParseElement (ctxt=ctxt@entry=0x55555588caa0) at parser.c:10069
#4  0x0000555555580c40 in xmlParseDocument (ctxt=ctxt@entry=0x55555588caa0) at parser.c:10841
#5  0x00005555555811e7 in xmlDoRead (ctxt=0x55555588caa0, URL=0x55555563e6e4 "noname.xml", encoding=<optimized out>,
    options=<optimized out>, reuse=0) at parser.c:15298
#6  0x00005555555696a8 in LLVMFuzzerTestOneInput (
    data=0x55555588d4a0 "<xml:veX:z:\020=\"1.0\" \200ncoding=\"ISO-8859-1\"?>\n<!DOCTYPE rss [\n<!--\n\n  Rich Site Summary (RSS) 91 official DTD, proposed.\n  *  RSS is an XML vocabulary for describing\022  metadata about websites, and enabli"..., size=7853)
    at target.c:28
#7  0x000055555556984e in main (argc=2, argv=0x7fffffffe158) at driver.c:37
```

Then I checked all our seeds to see if the function ```xmlTreeEnsureXMLDecl``` is covered. The answer is No.
Then how about the caller of the ```xmlTreeEnsureXMLDecl```, which is ```xmlSearchNs```. The answer is No.
Then how about ```xmlSAX2StartElementNs```

OK. From the below code snippet, because of the branch at Line 2356, the function of ```xmlSearchNs``` is not hit. 

```
18378: 2356:    if ((URI != NULL) && (ret->ns == NULL)) {
    #####: 2357:        ret->ns = xmlSearchNs(ctxt->myDoc, parent, prefix);
    #####: 2358:  if ((ret->ns == NULL) && (xmlStrEqual(prefix, BAD_CAST "xml"))) {
    #####: 2359:      ret->ns = xmlSearchNs(ctxt->myDoc, ret, prefix);
```

Now, the question is: why branch at Line 2356 is not flipped.

I check all the seeds we have to see if any of the seed has ```URI != NULL```, the answer is no. Also, this is a concrete branch.

Then I check the caller for the ```xmlSAX2StartElementNs``` function, as below

```
306888: 9700:    if ((ctxt->sax != NULL) && (ctxt->sax->startElementNs != NULL) &&
   153444: 9701:  (!ctxt->disableSAX)) {
    18378: 9702:  if (nbNs > 0)
    #####: 9703:      ctxt->sax->startElementNs(ctxt->userData, localname, prefix,
    #####: 9704:        nsname, nbNs, &ctxt->nsTab[ctxt->nsNr - 2 * nbNs],
        -: 9705:        nbatts / 5, nbdef, atts);
        -: 9706:  else
    18378: 9707:      ctxt->sax->startElementNs(ctxt->userData, localname, prefix,
        -: 9708:                    nsname, 0, NULL, nbatts / 5, nbdef, atts);
```

The ```nsname``` is passed as ```URI```. For all the calls to ```startElementNs```, we have ```nsname``` equal to zero. We do have seeds where nsname is nonzero, but then ```ctxt->disableSAX``` is zero
