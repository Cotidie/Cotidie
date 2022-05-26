# 패키지 컨벤션
|MVVM 패턴            |  아키텍처 상세 |
|:--------------------:|:------------------:|
|![클린 아키텍처](https://user-images.githubusercontent.com/51331195/157505761-e79cc80a-1301-4492-8441-dd3d7f21234b.png) | ![Flow](https://user-images.githubusercontent.com/51331195/157505816-10b27794-9bb6-4676-a186-3bee81683060.png) |

이 문서에서는 안드로이드에 권장하는 프로젝트 구조를 설명합니다. 앞으로도 계속 보완해나갈 예정입니다.

## 레이어 패턴
 Martin Fowler가 제시한 Presentation-Domain-Data 레이어 구분을 따르고자 합니다. 이 구조는 UI 요소가 존재하는 다양한 소프트웨어에 적용할 수 있는 기본적인 구조입니다. 관심사를 세 가지 큰 영역으로 구분함으로써 클래스나 로직의 위치를 쉽게 식별할 수 있고, 개발할 때에는 한 영역에만 집중할 수 있게 됩니다.

 ### Presentation
  : UI와 관계된 요소를 모두 담습니다. View, ViewModel, Navigation 모듈 등이 여기에 속합니다.

 ### Domain
  : UI와 데이터 영역 이외의 모든 코드가 여기에 담깁니다. 순수 언어(코틀린 등)으로 짜인 컨트롤러 등이 여기에 속합니다.

 ### Data
  : Persistence(영속성)과 연관이 깊은 요소를 담습니다. DB, Network, DTO 등이 여기에 속합니다.

## 프로젝트 구조
```m
|-app
    |- _constants         // Color, Size 등 UI 요소
    |- _enums             
    |- data
        |- room
            |- dao
            |- entity
        |- network
            |- dto
            |- api        
        |- storage        // SharedPreference, Datastore 등
    |- domain
        |- interfaces
        |- mapper         // data <-> domain 맵퍼
        |- model
        |- repository
        |- utils          // 컨트롤러 등
    |- presentation
        |- mapper         // domain <-> presentation 맵퍼
        |- model
        |- navigation
        |- component      // 공통으로 사용할 UI 컴포넌트 (공유 컴포저블 등)
        |- view
            |- <화면1>
            |- <화면2>
            |- ...
        |- viewmodel
```

### Mapper
 동일한 개념의 자료구조, 클래스여도 레이어마다 다르게 정의되어 있을 수 있습니다. 맵퍼는 주로 이 자료구조, 클래스의 레이어 이동을 담당합니다. 구현을 용이하게 하기 위해 아래와 같이 기본 인터페이스와 추상 클래스를 둘 수 있습니다.  
#### Interface
```kotlin
// 패키지: domain.interfaces
// 파일명: Mapper.kt
interface Mapper<I, O> {
    fun map(input: I) : O
}
```

```kotlin
// 패키지: domain.interfaces
// 파일명: ListMapper.kt
abstract class ListMapper<I, O>(
    private val mapper: Mapper<I, O>
) {
    override fun map(input: List<I>): List<O> {
        return input.map { mapper.map(it) }
    }
}
```

#### Mappers
맵퍼를 활용할 때에는 같은 범주는 하나로 묶어서 사용합니다. 예를 들어 Link -> LinkEntity, LinkEntity -> Link, List\<Link\>, List\<LinkEntity\>와 같이 4개의 맵퍼를 이용하고자 LinkMappers를 정의하고, 편의를 위해 메소드 오버로딩을 십분 활용합니다.
* 개별적인 Mapper 구현체는 private으로 숨깁니다.
```kotlin
// 패키지: domain.mapper
// 파일명: LinkMapper
@Singleton
class LinkMappers @Inject constructor(
    private val linkMapper : LinkToEntity,
    private val entityMapper : EntityToLink,
    private val linkListMapper : LinkListMapper,
    private val entityListMapper : LinkEntityListMapper,
) {
    fun map(link: ILink) : LinkWithTags = linkMapper.map(link)
    fun map(link: LinkWithTags) : ILink = entityMapper.map(link)
    @JvmName("LinkListToEntities")
    fun map(links: List<ILink>) : List<LinkWithTags> = linkListMapper.map(links)
    @JvmName("EntitiesToLinkList")
    fun map(links: List<LinkWithTags>) : List<ILink> = entityListMapper.map(links)
}

private class LinkToEntity: Mapper<ILink, LinkWithTags> {
    override fun map(input: ILink): LinkWithTags {
        ...
    }
}

private class EntityToLink: Mapper<LinkWithTags, ILink> {
    override fun map(input: LinkWithTags): ILink { 
        ... 
    }
}

private class LinkListMapper(
    linkMapper: LinkToEntity = LinkToEntity(),
): ListMapper<ILink, LinkWithTags>(linkMapper)

private class LinkEntityListMapper(
    entityMapper: EntityToLink = EntityToLink(),
): ListMapper<LinkWithTags, ILink>(entityMapper)
```

## :link: 참고
* [Presentation-Domain-Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)
