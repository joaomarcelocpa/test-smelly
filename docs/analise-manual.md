# Análise manual de Test Smells (`userService.smelly.test.js`)

| # | Smell | Onde | Risco |
|---|-------|------|-------|
| 1 | **Lógica condicional** (`for` + `if/else` com `expect` dentro) | teste "deve desativar usuários se eles não forem administradores" | O teste fica difícil de ler e, se o ramo não executar, o `expect` nunca roda e o teste passa em falso. |
| 2 | **Teste sem falha garantida** (`try/catch` com `expect` só no `catch`) | teste "deve falhar ao criar usuário menor de idade" | Se a validação for removida e nenhuma exceção for lançada, o teste continua verde e esconde o bug. |
| 3 | **Eager Test** (vários comportamentos em um teste) | "criar e buscar" e o teste do loop (desativar comum + admin) | Quando falha, não se sabe qual comportamento quebrou; nome vago. |
| 4 | **Teste frágil** (acoplado ao formato exato do relatório) | teste do relatório (`\n`, ordem, `startsWith`) | Qualquer mudança cosmética no relatório quebra o teste sem haver bug. |
| 5 | **Teste ignorado / vazio** | `test.skip` sem asserts | Cobertura enganosa: o caso "sem usuários" nunca é verificado. |
| 6 | **Cobertura faltante** | — | Não há teste para campos obrigatórios, `getUserById` inexistente e desativar id inexistente. |

## Comparação com o ESLint (1ª execução)

Veja `docs/eslint-primeira-execucao.txt`: o `jest/no-conditional-expect` pegou os smells 1 e 2
(linhas 44, 46, 49, 73), e `jest/no-disabled-tests` + `jest/expect-expect` pegaram o 5 (linha 77).
Os smells 3, 4 e 6 são semânticos e **não** são detectados pelo ESLint, apenas pela análise manual.

## Antes → Depois (exemplo para o relatório)

**Antes** (loop + if/else + expects condicionais):
```js
for (const user of todosOsUsuarios) {
  const resultado = userService.deactivateUser(user.id);
  if (!user.isAdmin) { expect(resultado).toBe(true); /* ... */ } else { expect(resultado).toBe(false); }
}
```
**Depois** (4 testes AAA, um comportamento cada): ver bloco `describe('deactivateUser')` em
`test/userService.clean.test.js`.
