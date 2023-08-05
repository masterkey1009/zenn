---
title: "SpringBootでREST APIを作成してみる"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [java, Spring Boot]
published: false
---
SpringBoot 2.3以降は、dependenciesにspring-boot-starter-validationを追加する必要がある。
2.2以下はspring-boot-starter-webにjavax.validation*のライブラリが含まれていたようですが、2.3からは別になりました。

https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes#validation-starter-no-longer-included-in-web-starters


### JPAアノテーション
`@Entity`
JPAのEntityであることを示す。

`@Table`
テーブル名を指定する。`table`名とclass名が同じ場合は省略可能。

`@Column`
テーブルの列名を指定する。

### Lombokアノテーション
`@NoArgsConstructor`
引数なしのコンストラクターを自動で生成する。
JPAのEntityのライフサイクル上、引数なしのコンストラクターが必要。
