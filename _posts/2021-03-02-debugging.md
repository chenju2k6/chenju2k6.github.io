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


