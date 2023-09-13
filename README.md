# isort-issue
Demonstrate a subtlety of isort's firstparty/thirdparty classification

To see the issue, look at `src/python/foo/bar/bar.py` as we run 
with various cli args (the checked-in file has the correct formatting):

```bash
$ pants fmt :: && cat src/python/foo/bar/bar.py 
18:58:07.15 [INFO] Completed: Format with isort - isort made no changes.

✓ isort made no changes.
# Third Party
import colors

# First Party
import foo.qux.qux

$ pants fmt src/python/foo/bar/bar.py && cat src/python/foo/bar/bar.py 
19:03:05.42 [WARN] Completed: Format with isort - isort made changes.
  src/python/foo/bar/bar.py

+ isort made changes.
# Third Party
import colors
import foo.qux.qux
```

This is because `foo.qux` isn't in the sandbox, and isort detects that foo is a namespace package,
and therefore treats things in foo, but outside foo.bar, as thirdparty (this is presumably
to support the case of `foo.qux` living in a separate repo than `foo.bar` and so being thirdparty to it).

In this case bringing in direct deps will solve the problem. In fact, even bringing in any file in
`foo` that proves to isort that `foo` isn't a namespace package will do:

```bash
$ pants fmt src/python/foo/bar/bar.py src/python/foo/baz.py && cat src/python/foo/bar/bar.py 
19:08:22.53 [WARN] Completed: Format with isort - isort made changes.
  src/python/foo/bar/bar.py

+ isort made changes.
# Third Party
import colors

# First Party
import foo.qux.qux
```

This does get subtle when `foo.qux` is in fact in a different repo and users want it to be 
thirdparty in `foo.bar`. But in that case `foo` must be a namespace package, and so 
the right thing should happen!
