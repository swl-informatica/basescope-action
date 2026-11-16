# baseScope: resultados de testes

GitHub Action da Base2 para o Assessment de Desenvolvimento com IA. Ela lê, no próprio runner do
seu CI, os relatórios de cobertura e de teste de mutação, reduz cada um a números por arquivo
(linhas e ramos cobertos; mutantes mortos, sobreviventes e sem cobertura) e envia só esses
números ao baseScope.

- **Nenhum código sai do runner.** Formatos que embutem o código-fonte (como o relatório do
  Stryker) são reduzidos antes do envio, e o baseScope repete a redução ao receber.
- **Nenhum segredo a configurar.** A autenticação usa o token OIDC do próprio workflow; o
  baseScope confere o repositório e só aceita os que estão no escopo do seu assessment.

## Formatos aceitos

| Tipo      | Formatos                                          |
| --------- | ------------------------------------------------- |
| Cobertura | LCOV (`lcov.info`), Cobertura (XML), JaCoCo (XML) |
| Mutação   | Stryker (`mutation.json`), PIT (`mutations.xml`)  |

## Uso

O workflow precisa da permissão `id-token: write` para gerar o token OIDC.

```yaml
name: Testes
on:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write

jobs:
  testes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
      - run: npm ci
      - run: npm test -- --coverage
      - run: npx stryker run
      - uses: swl-informatica/basescope-action@v1
        with:
          url: ${{ vars.BASESCOPE_URL }}
          cobertura: coverage/lcov.info
          mutacao: reports/mutation/mutation.json
```

| Entrada                | Descrição                                                      |
| ---------------------- | -------------------------------------------------------------- |
| `url`                  | Endereço de recebimento informado pela Base2.                  |
| `cobertura`            | Caminhos ou padrões dos relatórios de cobertura, um por linha. |
| `mutacao`              | Caminhos ou padrões dos relatórios de mutação, um por linha.   |
| `falhar-sem-relatorio` | `true` para falhar quando nenhum relatório for encontrado.     |

O teste de mutação costuma ser lento; se preferir, rode-o num workflow agendado separado e
envie só a mutação.
