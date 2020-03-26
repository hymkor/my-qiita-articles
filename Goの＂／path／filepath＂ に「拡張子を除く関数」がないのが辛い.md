---
title: Goの"/path/filepath" に「拡張子を除く関数」がないのが辛い
tags: Go
author: zetamatta
slide: false
---
まぁ、自分で書いても数行なんですけどね

```without.go
func withoutExt(fname string) string {
	ext := filepath.Ext(fname)
	return fname[:len(fname)-len(ext)]
}
```

