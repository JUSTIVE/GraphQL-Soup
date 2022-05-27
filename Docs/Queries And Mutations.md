# 쿼리와 뮤테이션

이 페이지에서는 GraphQL 서버에 어떻게 쿼리하는지 배울 수 있습니다.

## 필드

가장 간단하게, GraphQL은 객체의 구제적인 필드를 요청하는 것입니다. 다음의 매우 간단한 쿼리와 우리가 얻을 수 있는 결과를 봅시다.

```gql
{
  hero {
    name
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

쿼리가 결과와 완전히 동일한 형태를 가지고 있다는 것을 바로 확인할 수 있습니다. 이는 GraphQL의 핵심으로, 우리가 항상 예상한 값들만 받을 수 있고, 서버는 클라이언트가 어떤 것들을 요청할지 정확히 알고 있기 때문에 가능합니다.

필드 `name`은 `String` 타입을 반환하고, 이 경우에는 `"R2-D2"`에 해당합니다.

위의 예제에서, 우리는 `hero`의 이름을 요청했고, `String`을 받았지만, 필드는 객체를 의미할 수도 있습니다. 이 경우, 그 객체의 `sub-selection`을 만들 수 있습니다. GraphQL 쿼리는 관련된 객체와 그들의 필드를 순회하여, 클라이언드들이 관련된 많은 데이터들을 고전적인 REST처럼 여러 개의 요청들을 순회하는 대신 하나의 요청에 처리할 수 있게 합니다.

```gql
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

위의 예제에서, `friends` 필드가 아이템의 배열을 반환하는 것에 주목하세요. GraphQL 쿼리는 하나의 항목이나 여러개의 항목이나 똑같은 외형을 가지고 있으나 우리는 스키마에 정의된 것에 의해 어떤 것을 받을 지 예상할 수 있습니다.

## 인자

만약 우리가 할 수 있는 것이 객체와 그들의 필드를 순회할 수 있는 것 뿐이라도, GraphQL은 이미 데이터를 가져오는 데에 충분히 유용한 언어일 것입니다. 그러나 인자를 넘겨줄 수있는 능력을 더한다면, 더욱 흥미로워집니다.

```gql
{
  human(id: "1000") {
    name
    height
  }
}
```

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

REST의 시스템에서는 하나의 인자 집합(쿼리 인자와 요청의 URL 세그먼트)만을 넘길 수 있었습니다. 그러나 GraphQL에서는 모든 필드와 중첩된 객체는 각각의 고유한 인자의 집합을 가질 수 있습니다. 이는 GraphQL을 복수의 API 페치를 만드는 데에 온전한 대체품으로 만듦니다. 심지어 스칼라 필드에 값을 넘김으로써 서버측의 데이터 변환을 각각의 클라이언트별로 나누어 하는 대신 한번에 수행할 수 있습니다.

```gql
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

인자는 다양한 종류의 타입이 될 수 있습니다. 위의 예제의 경우, 우리는 열거 타입을 사용했습니다. GraphQL은 타입들의 집합들이 딸려오나, GraphQL 서버는 전송 포맷으로 시리얼화 되어질 수 있는 한에서 사용자 정의 타입 또한 선언할 수 있습니다.

## 별칭

예리한 눈을 가지고 있다면, 결과 객체 필드가 쿼리의 필드 이름과 일치하지만, 인자가 포함되지 않기 때문에 다른 인자를 사용하여 동일한 필드를 쿼리할 수 없다는 것을 알 수 있을 것입니다.
별칭을 이용하면 필드 결과의 이름을 원하는 이름으로 사용할 수 있습니다.

```gql
{
  empireHero: hero(episode:EMPIRE){
    name
  }
  jediHero : hero(episode:JEDI){
    name
  }
}
```

```json
{
  "data":{
    "empireHero":{
      "name":"Luke Skywalker"
    },
    "jediHero":{
      "name":"R2-D2"
    }
  }
}
```
위의 예제에서, 두 hero 필드가 이름이 겹치지만, 우리는 다른 두 별칭을 이용함으로써 하나의 요청에 두 결과를 받을 수 있습니다.

## 프라그먼트

두 영웅들을 그들의 친구들과 함께 나란히 볼 수 있는 완전히 복잡한 페이지가 앱에 있다고 생각해 봅시다.
이런 쿼리는 우리가 필드들을 각각의 비교마다 반복해야 하기 때문에 순식간에 복잡해진다고 생각할 수 있을 것입니다.

이것이 GraphQL이 fragments라는 재사용 가능한 단위들을 가지고 있는 이유입니다. 프라그먼트는 필드들의 조합을 구성하여, 필요한 쿼리에 넣어 사용할 수 있게 합니다.

```gql
{
  leftComparison: hero(episode:EMPIRE){
    ...comparisonFields
  }
  rightComparison: hero(episode:JEDI){
    ...comparison
  }
}
fragment comparisonFields on Character {
  name 
  appearIn
  friends {
    name
  }
}
```

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

위의 쿼리에서 모든 필드들이 반복되었다면 꽤 반복적이었을 것이라는 것을 확인할 수 있습니다.
프라그먼트의 개념은 복잡한 애플리케이션 데이터 요청을 더 작은 청크로 분할하는 데에 사용합니다. 
특히 여러 UI 구성 요소를 다른 프래그먼트와 결합하여 하나의 초기 데이터 가져오기로 결합해야 할 때 그렇습니다.

### 프라그먼트에서 변수 사용하기

쿼리 혹은 뮤테이션에서 정의된 변수를 프라그먼트에서 접근하는 것이 가능합니다.

```gql
query HeroComparison($first:Int = 3){
  leftComparison: hero(episode:EMPIRE){
    ...comparisonFields
  }
  rightComparison: hero(episode:JEDI){
    ...comparisonFields
  }
}
fragment comparisonFields on Character{
  name
  friendsConnection(first:$first){
    totalCount
    edges{
      node{
        name
      }
    }
  }
}
```

## 운영 이름

지금까지, 우리는 `query`키워드와 쿼리 이름을 모두 생략하는 축약형 문법을 사용했지만, 프로덕션 앱에서는 코드를 덜 모호하게 만드는 데 사용하는 것이 유용합니다.

다음의 예제는 `query` 키워드로 _운영 타입_ 과 `HeroNameAndFrieds` 를 _운영 이름_ 으로 가지는 예시입니다.
```gql
query HeroNameAndFriends {
  hero{
    name
    friends {
      name
    }
  }
}
```

_운영 타입_ 은 수행하려는 연산의 의도를 설명하는 `query`, `mutation`, 혹은 `subscription`와 같은 것입니다.
축약 문법을 사용하지 않는다면, 운영 이름을 사용하는 것이 요구됩니다.
이 경우 연산의 이름 부여나, 변수 정의를 할 수 없습니다.

## 변수

지금까지 모든 변수를 쿼리 문자열 안에 써 왔습니다. 그러나 대부분의 애플리케이션에서는, 인자 필드들은 동적일 것입니다.

이 동적 인자들을 직접적으로 쿼리 문자열에 넘기는 것은 클라이언트측에서 쿼리 문자열을 런타임에 동적으로 제어해서 GraphQL 특화 형식으로 직렬화를 해야 하기 때문에 좋은 생각이 아닙니다.
대신, GraphQL은 쿼리에서 동적 값들을 분해하고, 별개의 사전으로 전달하는 1급 방법이 있습니다.
이러한 값들은 변수라 불립니다. 

변수를 다룰 때에, 다음의 3가지가 필요합니다.

1. 쿼리의 정적 값을 `$변수명`으로 바꿉니다.
2. 쿼리에서 받아들이는 변수 중 하나로 `$변수명`을 선언합니다.
3. `변수명: 값`을 별개의 전송 특화된 사전 변수로 전달합니다.(일반적으로 JSON)

이는 다음과 같을 것입니다. 
```gql
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

이제 클라이언트 온전히 새 쿼리를 만들 필요 없이 단순히 다른 변수를 전달할 수 있습니다.
이는 또한 어떤변수들이 동적으로 될 지 표시하는 좋은 방법입니다.

## 변수 선언

변수 선언부는 쿼리 위의 `($episode: Episode)`와 같이 생긴 부분입니다. 
이는 정적 타입 언어에서 함수에 넘겨지는 인자와 같이 동작합니다.
이는 모든 `$`로 시작하고 타입들이 적히는 변수들을 나타냅니다. 

모든 선언된 변수들은 반드시 스칼라, 열거형, 혹은 입력 객체 타입이어야 합니다.
따라서 만약 복잡한 객체를 필드로 넘기고 싶다면 서버측에서 어떤 입력 타입이 대응되는 지 알아야 합니다.

변수 선언은 선택적이거나 필수일 수 있습니다. 위의 예제에서 만약 Episode 옆에 `!` 가 없다면, 이것은 선택적입니다.
하지만 만약 변수를 전달하는 필드에 널이 아닌 인자가 필요한 경우 변수도 필요해야 합니다.

## 기본값

변수에 타입 선언부 이후에 기본값을 지정할 수 있습니다.

```gql 
query HeroNameAndFriends($episode: Episode = JEDI){
  hero(episode:$episode){
    name
    friends {
      name
    }
  }
}
```

만약 모든 변수들에 기본값이 제공되었다면, 쿼리를 변수를 넘기지 않고도 호출할 수 있습니다. 
만약 변수 사전 중 어떤 일부의 변수라도 넘겨졌다면 기본값을 덮어쓸 것입니다.

## 지시어

위에서 변수들이 어떻게 수동 보간문자열을 피하고 동적 쿼리를 만들게 하는 지 알아보았습니다.
인수에 변수를 넘기는 것은 이 문제를 해결하는 큰 부분이지만, 구조를 동적으로 바꾸고 변수를 이용해서 쿼리의 모양을 바꾸는 법도 필요할 수 있습니다.
예를 들어, 하나가 다른 것보다 많은 필드를 포함하고 있는 UI 컴포넌트가 요약과 상세보기를 가지고 있는 컴포넌트가 있다고 생각해 봅시다.

```gql
query Hero($episode :Episode, $withFriend:Boolean!){
  hero(episode:$episode){
    name
    friends @include(if:$withFriends){
      name
    }
  }
}
```

여기서 GraphQL에서 _지시어_ 라고 불리는 것들을 사용해야 합니다.
지시어는 필드나 프라그먼트 포함절에 붙어 있을 수 있으며, 서버가 원하는 방식으로 쿼리 실행에 영향을 줄 수 있습니다.
코어 GraphQL 스펙은 오로지 두 지시어를 지원하며, 이는 스펙을 준수하는 모든 GraphQL 서버에 구현체에서 지원되어야 합니다.

- @include(if:Boolean) 인자가 true일때만 이 필드를 포함
- @skip(if:Boolean) 인자가 true 이면 이 필드를 생략

지시어는 쿼리 문자열을 수동으로 조작해야 하는 상황들을 벗어날 때 유용합니다.
서버 구현체는 완전히 새로운 지시어를 선언하여 실험적인 기능들을 더했을 수도 있습니다.

## 뮤테이션

GraphQL이 데이터 페칭에 중점을 두고 있으나, 모든 온전한 데이터 플랫폼은 서버측의 데이터를 바꿀 방법도 필요합니다.

REST에서는, 모든 리퀘스트가 서버 측의 부작용을 가져올 수 잇으나, 컨벤션을 통해 `GET`을 통해 데이터를 바꾸는 것을 권장하지 않습니다.
GraphQL도 비슷합니다. 기술적으로 모든 쿼리들은 데이터 쓰기를 하게끔 구현할 수 있습니다.
반면, 쓰기를 유발하는 동작들을 명시적으로 mutation으로 구축하는 규약은 유용합니다.

쿼리와 마찬가지로, 만약 뮤테이션 필드가 객체 타입을 반환하다면, 중첩된 필드를 요청할 수 있습니다.
이는 갱신 이후의 새 상태나 객체를 페칭하는데에 유용할 것입니다.
다음의 간단한 뮤테이션을 보세요.

```gql
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

`createReview` 필드가 새로 만들어진 리뷰에서 stars 와 commentary필드를 반환하는 것을 볼 수 있습니다.
이는 있는 데이터를 변환할때에 특히 유용합니다. 예를 들어, 필드의 값을 증가시킬 때에, 우리는 값을 변화하고 새 값을 받아오는 것을 하나의 쿼리로 수행할 수 있습니다.

위의 예제에서 보듯, 이 예제에서 `review` 변수는 스칼라 값이 아닙니다.
이는 입력 객체 타입으로, 인자로 넘겨질 수 있는 특수한 객체 타입입니다.
스키마 페이지에서 보다 자세히 다룹니다.

### 뮤테이션의 다중 필드

뮤테이션은 쿼리처럼 여러 필드를 포함할 수 있습니다.
이름 말고도 쿼리와 뮤테이션을 구분하는 중요한 차이가 있습니다.

쿼리는 병렬로 수행되지만, 뮤테이션은 필드를 순차적으로 실행합니다.

이는 두 `incrementCredits` 뮤테이션을 하나의 요청에 보냈을 때에 첫 번째는 두번째가 실행되기 이전에 끝나는 것이 보장되고, 우리 스스로 경쟁 상태에 놓이게 되는 것을 방지합니다.

## 인라인 프라그먼트

많은 다른 타입 시스템들처럼, GraphQL 스키마도 인터페이스와 유니온 타입을 선언할 수 있는 능력을 포함하고 있습니다.

만약 인터페이스 혹은 유니온 타입을 반환하는 쿼리가 있다면, _인라인 프라그먼트_ 를 사용해서 견고한 타입에 기반한 데이터들에 접근해야 할 것입니다.
이는 예제로 보는 것이 가장 간단합니다.

```gql
query HeroForEpisode($ep:Episode!){
  hero(episode:$ep){
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```
이 쿼리에서, `hero` 필드는 `Character`타입의 필드를 반환합니다.
이는 episode 인자에 따라 `Human` 혹은 `Droid`가 될 수 있습니다.
직접 선택에서는 `name`과 같이 Character 인터페이스에서 존재하는 필드를 요청할 수 있습니다.

견고한 타입에서 필드를 요청하기 위해서는, `인라인 프라그먼트`와 타입 조건을 사용해야 합니다.
첫 프라그먼트가 `... on Driod` 로 표기되었기 때문에, `primaryFunction`필드는 오직 `hero`에서 반환 타입이 `Droid`일때만 실행될 것입니다.
`Human`타입의 `height` 필드도 이와 동일합니다.

이름있는 프라그먼트는 항상 타입이 딸려있기 때문에 이름있는 프라그먼트도 이와 같이 선언될 수 있습니다.

## 메타 필드

GraphQL 서비스에서 어떤 타입을 받을 지 모르는 상황에서, 클라이언트에서 이 데이터를 어떻게 다룰 지 결정해야 하는 상황이 있다고 합시다.
GraphQL에서는 메타 필드인 `__typename` 필드를 요청할 수 있게 합니다.
이 필드는 아무때나 해당 객체 타입의 이름을 쿼리합니다.