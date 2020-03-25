# Git



## Patch

#### Applying the Patch

```shell
#shows you the stats about what it’ll do
git apply --stat fix_empty_poster.patch

#test the patch before you actually apply it
git apply --check fix_empty_poster.patch

#apply directly
git apply fix_empty_poster.patch

#apply
git am --signoff < fix_empty_poster.patch
```

