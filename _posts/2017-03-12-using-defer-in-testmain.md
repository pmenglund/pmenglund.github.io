---
layout: post
title:  "Using defer in TestMain"
date:   2017-03-12 14:00:00
categories: golang, testing
---
If you're using `TestMain` you've probably read that you can't use `defer` as you need to use `os.Exit(m.Run())` to exit from `TestMain`. The reason for that is that `os.Exit()` doesn't honor `defer` statements.

There is an easy way around that, just wrap your code in a function where you use defer. For example, if your code looks like this

```go
func TestMain(m *testing.M) {
  db := db.Connect(...)
  db.Create(...)

  code := m.Run()

  db.Delete(...)
  db.Close()
  os.Exit(code)
}
```
Then you just extract everything into a `testMainWrapper()` that returns the return code for `os.Exit()`
```go
func TestMain(m *testing.M) {
  os.Exit(testMainWrapper(m))
}

func testMainWrapper(m *testing.M) int {
  db := db.Connect(...)
  db.Create(...)
  defer func() {
    db.Delete(...)
    db.Close()
  }
  return m.Run()
}
```
