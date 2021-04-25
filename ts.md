``` ts
// 类型推断
type ApiType<Req, Res> = {}
type Type = {
  a: ApiType<{ test: string }, { list: number[] }>
  b
}

function create<T>(args: T) {
  return {} as {
    [P in keyof T]: (data?: T[P] extends ApiType<infer Req, any> ? Req : any)
      => T[P] extends ApiType<any, infer Res> ? Res : any
  }
}

let x = create({} as Type)
/*
(property) a: (data?: {
    test: string;
}) => {
    list: number[];
}
*/
x.a()

// (property) b: (data?: any) => any
x.b()
```
