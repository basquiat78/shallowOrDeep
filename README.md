
# shallowOrDeep

call by value, call by reference를 정리하다가 여기까지 오게 되었다.

무엇인고 하면 바로 shallow copy와 deep copy이다.

# 먹는건가요??

넹 먹는겁니다.

일단 이 주제를 가지고 흔적을 남기는 이유는 위에서 언급했던 call by value, call by reference때문이다.

자바를 주제로 하자면 (아마도 javascript같은 다른 언어도 비슷하겠지만) 타입에 대한 어느 정도의 개념이 있어야 한다.

Primitive Type -> 원시 자료형이라고 한다. 예를 들면 String, int, double, long, char을 생각하면 된다. 
                  이런 경우에는 call by value로 작동한다.


Reference Type -> 참조 자료형이라고 한다. 
                  List같은 Collection이나 Json같은 종류나 객체를 보통 Reference Type이라고 하는데 이것때문에 자바에서는 call by reference에 대한 오해를 불러 온다.

뭐 제임스 고슬링옹께서 설계한 의도가 있겠지만 그냥 여기서는 가볍게 지나가보자. 

# 우리가 가장 간단하게 할 수 있는 단순 복사의 예제

일단 브라우저에서 개발자 도구를 열어서 콘솔에서 코드를 짜거나 [repl](https://repl.it/) 에서 프로젝트를 nodejs로 생성해서 진행해 볼 수 있다.

repl.it에서 하는 것을 추천한다.

자바도 똑같기 때문에 자바로 하든 nodeJs로 하든 상관없다.

```
let original = "a";
let copied = original;
console.log(original, copied)

original = "b";
copied = "c";
console.log('original를 b로 변경한 이후')
console.log('copied c로 변경한 이후')
console.log(original, copied)

```
정말 간단하게 코드를 짜서 실행하면 우리는 예상되는 다음 결과를 얻게 된다.

```
a a
original를 b로 변경한 이후
copied c로 변경한 이후
b c
```

저 위에서는 스트링 타입이지만 number으로 해도 결과는 마찬가지이다. 
Primitive Typ의 경우에는 copied라는 변수에 original를 대입할 때는 pass by value처럼 값이 복사되서 세팅된다.
그리고 이 두 개를 변경해도 서로 다르게 동작하게 된다.

하지만 만일 Reference Type를 저렇게 사용하게 되면 어떤 일이 생길까..

```
let original = {a: 1};
let copied = original;
console.log(original, copied)

original.a = 2;
console.log('original 변경한 이후')
console.log(original, copied)
copied.a = 3;
console.log('copied 변경한 이후')
console.log(original, copied)

```

결과는요??

```
{ a: 1 } { a: 1 }
original 변경한 이후
{ a: 2 } { a: 2 }
copied 변경한 이후
{ a: 3 } { a: 3 }

```

어라? original을 변경하든 copied를 변경하든 전부 똑같이 바껴버리는 이 상황.

근데 이런 경우도 한번 생각해 보자.

```
let original = [{a: 1}];
let copied = original;
console.log(original, copied)

copied.push({a: 2});
console.log(original, copied)

original.push({a: 3});
console.log(original, copied)

```

결과는?

```
[ { a: 1 } ]                        [ { a: 1 } ]
[ { a: 1 }, { a: 2 } ]              [ { a: 1 }, { a: 2 } ]
[ { a: 1 }, { a: 2 }, { a: 3 } ]    [ { a: 1 }, { a: 2 }, { a: 3 } ]

```

오호라? 이 경우는 pass by reference처럼 해당 객체의 주소를 그냥 넘겨 받는건가보네?라고 생각이 든다.

결국 원복 데이터를 저런식으로 단순 복사해서 또 다른 비지니스 로직을 태운다는 것은 이슈가 발생할 소지가 다분하다는 것이다.

자바라면 clone을 이용해서 복사하거나 javascript라면 lodash를 이용해서 복사를 해서 쓴다고 해보자.

# shallow copy (얕은 복사) 

```

const _ = require('lodash');

let original = [{a: 1}];
let copied = _.clone(original);
console.log(original, copied)

copied.push({a: 2});
console.log(original, copied)
 

```

위와 같은 경우를 생각해 보자.
1. 원복 데이터를 clone를 이용해서 복사하자.
2. 복사한 데이터에만 배열 하나를 추가해 보자.

결과는요?

```
[ { a: 1 } ]        [ { a: 1 } ]
[ { a: 1 } ]        [ { a: 1 }, { a: 2 } ]

```

오호! 하지만 다음과 같이 해보면 어떨까?

```
const _ = require('lodash');

let original = [{a: 1}];
let copied = _.clone(original);
console.log(original, copied)

copied.push({a: 2});
_.forEach(copied, (json, index) => json.a = index);
console.log(original, copied)

```

결과가 궁금하지? 아는 사람은 안궁금하겠지?

```

[ { a: 1 } ]    [ { a: 1 } ]
[ { a: 0 } ]    [ { a: 0 }, { a: 1 } ]

```   

어? 오호? 복사가 되긴 했지만 마치 자바의 '그것'처럼 작동하네?
결국 copied 새로 생성되서 복사되지만 객체 내부의 프로퍼티, 즉 Reference Type의 값들은 Reference라는 건가?

만일 위처럼 Json객체가 리스트로 있는 형태가 아닌 다음과 같은 형태라면?

```

const _ = require('lodash');

let original = ["a", "b", "c"];
let copied = _.clone(original);
console.log(original, copied)

copied.push("d");
for(let i = 0; i < copied.length; i++) {
  copied[i] = copied[i] + "_" + i;
}
console.log(original, copied)

```

결과는?

```

[ 'a', 'b', 'c' ]   [ 'a', 'b', 'c' ]
[ 'a', 'b', 'c' ]   [ 'a_0', 'b_1', 'c_2', 'd_3' ]

```

아하! 얕은 복사이긴 하지만 배열 내부의 프로퍼티는 Reference Type이 아닌 Primitive Type이라서 original의 배열 값들은 변경되지 않았다.

~~Reference Type이 잘못했네!~~ 


# deep copy (깊은 복사) 

위 예제에서 코드만 살짝 바꿔보자

```
const _ = require('lodash');

let original = [{a: 1}];
let copied = _.cloneDeep(original);
console.log(original, copied)

copied.push({a: 2});
_.forEach(copied, (json, index) => json.a = index);
console.log(original, copied)

```

결과는 다음과 같다.

```
[ { a: 1 } ]    [ { a: 1 } ]
[ { a: 1 } ]    [ { a: 0 }, { a: 1 } ]

```

# 라이브러리 없이 shallow copy, deep copy를 할 수 있나?

물론 찾아보면 할 수 있는 방법이 널려 있다.

하지만 굳이 생짜 코드로 짜야 할까? 

개발자라면 해야 하는지도 모른다. 

하지만 난 그렇고 싶지 않다. 

게으르고 싶단 말이야! 차라리 그 시간에 다른 것을 보겠어!

그 이유는 여러가지가 있는데 해당 객체를 iterate하면서 담겨진 프로퍼티가 Primitive Type인지 Reference Type인지를 체크해야 하고 만일 Reference Type이라면 또 iterate를 돌아야 한다.

자바라면 Reflection까지 가야할 껄?


쉽게 하자면 

자바스크립트의 경우에는 lodash가 있다. ~~이거하라고 만든 라이브러리 아니다~~~

자바의 경우에는 좀 다른데 org.apache.http.client.utils에 있는 CloneUtils를 사용할 수 있다.

하지만 이 경우에는  deep copy를 하기 위해서는 별도의 코딩을 짜야 한다.

POJO같은 객체라면 gson이나 잭슨이 아저씨의 라이브러리 api를 이용해 string으로 변환하고 그것을 객체로 다시 변환하는 간단한 방법이 있다. 

생각해 보면 이게 가장 확실하지 않을까?


# At A Glance

별거 아닌거 같지만 이 문제로 회사에서 데이터가 꼬여버린 사건이 최근에 있었다.

~~후배 동료님 힘내세요!! 괜찮아요. 저도 예전에 사고쳤었어요.~~

사실 저렇게 복사해서 쓸 일도 생각해보면 그다지 많지 않기도 했었고 하니...

하지만 아주 중요한 포인트인건 확실하다.
