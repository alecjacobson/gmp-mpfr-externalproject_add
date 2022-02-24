Build using rosetta on mac m1

```bash
cmake ../ -DCMAKE_OSX_ARCHITECTURES=x86_64
arch -x86_64 make
```
