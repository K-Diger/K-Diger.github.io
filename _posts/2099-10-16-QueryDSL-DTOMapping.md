---

title: QueryDSL 조회 값 DTO 와 매핑하기

date: 2022-10-16
categories: [QueryDSL]
tags: [DTO, Constructor, Bean, Setter]
layout: post
toc: true
math: true
mermaid: true

---

# QueryDSL과 DTO를 매핑하는 방법

총 4가지가 존재하는데

1. Projections.bean (프로퍼티 접근 Setter)
2. Projections.fields (필드 접근)
3. Projections.constructor (생성자 접근)
4. @QueryProjection (생성자 활용)

특징: DTO 생성자에 @QueryProjection어노테이션을 붙이면 자동으로 생성되고 실제 클래스가 선언한 생성자기반으로 작성됨. 따라서 제일권장됨.


## Projections.bean (프로퍼티 접근 Setter)

    /**
     *
     * @param pageable
     * @param searchValue
     * @return 검색결과 조회
     * 수정날짜 내림차순으로, 10개를 뽑아온다. (페이징)
     */
    @Override
    public List<SearchResultResponse> searchPost(Pageable pageable, String searchValue) {

        List<SearchResultResponse> response = queryFactory
                    .select(Projections.bean(SearchResultResponse.class,
                            productPost.id,
                            productPostFile.imageLink,
                            productPost.contactPlace, productPost.updatedAt,
                            productPost.title, productPost.price,
                            productPost.user.nickname, productPost.category))
                    .from(productPost)
                    .where(productPost.title.contains(searchValue))
                    .leftJoin(productPostFile).on(productPostFile.productPost.id.eq(productPost.id))
                    .orderBy(productPost.updatedAt.desc())
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetch();
    }

특징: DTO에 Setter 가 존재해야 사용할 수 있게 된다. 이 말인 즉슨 수정 과정을 거치기 때문에 불변 객체를 지향하고자 할 때 사용하지 않는 것이 좋다.

## Projections.fields (필드 직접 접근)

    /**
     *
     * @param pageable
     * @param searchValue
     * @return 검색결과 조회
     * 수정날짜 내림차순으로, 10개를 뽑아온다. (페이징)
     */
    @Override
    public List<SearchResultResponse> searchPost(Pageable pageable, String searchValue) {

        List<SearchResultResponse> response = queryFactory
                    .select(Projections.fields(SearchResultResponse.class,
                            productPost.id,
                            productPostFile.imageLink,
                            productPost.contactPlace, productPost.updatedAt,
                            productPost.title, productPost.price,
                            productPost.user.nickname, productPost.category))
                    .from(productPost)
                    .where(productPost.title.contains(searchValue))
                    .leftJoin(productPostFile).on(productPostFile.productPost.id.eq(productPost.id))
                    .orderBy(productPost.updatedAt.desc())
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetch();

        return response;
    }

특징: Setter 없이 사용가능한 방법이다.
필드에 직접 접근하여 값을 매핑하는 방법이다.

## 3. Projections.constructor


    /**
     *
     * @param pageable
     * @param searchValue
     * @return 검색결과 조회
     * 수정날짜 내림차순으로, 10개를 뽑아온다. (페이징)
     */
    @Override
    public List<SearchResultResponse> searchPost(Pageable pageable, String searchValue) {

        List<SearchResultResponse> response = queryFactory
                    .select(Projections.constructor(SearchResultResponse.class,
                            productPost.id,
                            productPostFile.imageLink,
                            productPost.contactPlace, productPost.updatedAt,
                            productPost.title, productPost.price,
                            productPost.user.nickname, productPost.category))
                    .from(productPost)
                    .where(productPost.title.contains(searchValue))
                    .leftJoin(productPostFile).on(productPostFile.productPost.id.eq(productPost.id))
                    .orderBy(productPost.updatedAt.desc())
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetch();

        return response;
    }

특징: 생성자 기반, 이때 바인딩 방식이용으로 생성자 넘기는 순서 달라질 수 있다.(DTO가 가지고있는 생성자이용X)
