<details>
  <summary>Реализация scope-функции let:</summary>

`inline fun <T, R> T.let(block: (T) -> R): R {`
`return block(this)`
`}`

</details>