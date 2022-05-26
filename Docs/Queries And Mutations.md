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