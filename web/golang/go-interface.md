# Go interface



## Go interface

Go version: 1.14.4

### 接口

#### eface 空接口

* 结构

```go
// runtime/runtime2.go
type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

所有类型的结构体都包含 \_type

```go
// Needs to be in sync with ../cmd/link/internal/ld/decodesym.go:/^func.commonsize,
// ../cmd/compile/internal/gc/reflect.go:/^func.dcommontype and
// ../reflect/type.go:/^type.rtype.
// ../internal/reflectlite/type.go:/^type.rtype.
type _type struct {
    size       uintptr
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32
    tflag      tflag
    align      uint8
    fieldAlign uint8
    kind       uint8 // 类型编号
    // function for comparing objects of this type
    // (ptr to object A, ptr to object B) -> ==?
    equal func(unsafe.Pointer, unsafe.Pointer) bool
    // gcdata stores the GC type data for the garbage collector.
    // If the KindGCProg bit is set in kind, gcdata is a GC program.
    // Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
    gcdata    *byte // GC
    str       nameOff
    ptrToThis typeOff
}
```

* 流程概述
  * 编译器根据赋值类型调用 conTx 或 conT2E 
  * conTx 后编译器组装 eface 结构，conT2E 会将组装完整的 eface 返回 ，[参考](https://github.com/golang/go/commit/5848b6c9b854546473814c8752ee117a71bb8b54)

#### iface

* 结构

```go
// runtime/runtime2.go
type iface struct {
    tab  *itab // interface type and data type
    data unsafe.Pointer
}
```

```go
type iface struct { // `iface`
    tab *struct { // `itab` 40 bytes
        inter *struct { // `interfacetype`
            typ struct { // `_type`
                size       uintptr
                ptrdata    uintptr
                hash       uint32
                tflag      tflag
                align      uint8
                fieldalign uint8
                kind       uint8
                alg        *typeAlg
                gcdata     *byte
                str        nameOff
                ptrToThis  typeOff
            }
            pkgpath name
            mhdr    []struct { // `imethod`
                name nameOff
                ityp typeOff
            }
        }
        _type *struct { // `_type`
            size       uintptr
            ptrdata    uintptr
            hash       uint32
            tflag      tflag
            align      uint8
            fieldalign uint8
            kind       uint8
            alg        *typeAlg
            gcdata     *byte
            str        nameOff
            ptrToThis  typeOff
        }
        hash uint32
        _    [4]byte
        fun  [1]uintptr
    }
    data unsafe.Pointer
}
```

* 流程概述
  * 编译器根据赋值类型调用 conTx 或 conT2I 
  * conTx 后编译器组装 iface 结构，conT2I 会将组装完整的 iface 返回 ，[参考](https://github.com/golang/go/commit/5848b6c9b854546473814c8752ee117a71bb8b54)

### 总结

测试版本的 go 在生成接口为统一的逻辑，`runtime` 包的函数会返回 `data unsafe.Pointer`，编译过程使用编译器已有类型信息和 `runtime` 中返回的 `data unsafe.Pointer` 组装成完整接口。

> convT2E16 and other specialized type-to-interface routines accept a type/itab argument and return a complete interface value. However, **we know enough in the routine to do without the type. And the caller can construct the interface value using the type.** Doing so shrinks the call sites of ten of the specialized convT2x routines. **It also lets us unify the empty and non-empty interface routines.**

