# 🤝 Contribuindo

Obrigado pelo interesse em contribuir com o Visualizador de Parciais Harmônicos!

## Como contribuir

### Reportando bugs

1. Verifique se o bug já não foi reportado em [Issues](../../issues)
2. Crie uma nova issue usando o [template](.github/ISSUE_TEMPLATE.md)
3. Inclua informações sobre seu navegador e sistema operacional
4. Se possível, inclua screenshots ou gravações de tela

### Sugerindo melhorias

1. Abra uma issue com a tag `enhancement`
2. Descreva a funcionalidade desejada e o caso de uso
3. Se possível, inclua mockups ou referências visuais

### Enviando código

1. **Fork** o repositório
2. Crie uma branch para sua feature:
   ```bash
   git checkout -b feature/minha-feature
   ```
3. Faça seus commits com mensagens claras:
   ```bash
   git commit -m "feat: adiciona exportação de dados em CSV"
   ```
4. Push para sua branch:
   ```bash
   git push origin feature/minha-feature
   ```
5. Abra um **Pull Request** descrevendo as mudanças

## Convenções de commit

Utilizamos [Conventional Commits](https://www.conventionalcommits.org/):

| Prefixo | Uso |
|---------|-----|
| `feat:` | Nova funcionalidade |
| `fix:` | Correção de bug |
| `docs:` | Alteração na documentação |
| `style:` | Formatação, CSS (sem alteração de lógica) |
| `refactor:` | Refatoração de código |
| `perf:` | Melhoria de performance |
| `test:` | Adição ou correção de testes |

## Diretrizes de código

- O aplicativo é **single-file** (`index.html`). Mantenha essa estrutura a menos que haja uma razão forte para modularizar.
- **Zero dependências externas** (exceto Google Fonts). Não adicione bibliotecas sem discussão prévia.
- Priorize **compatibilidade entre navegadores** — teste em Chrome, Firefox e Safari.
- Comente blocos complexos de processamento de sinal (FFT, autocorrelação, etc.).
- Mantenha a interface em **português brasileiro** como idioma principal.

## Ambiente de desenvolvimento

```bash
# Clone seu fork
git clone https://github.com/SEU-USUARIO/harmonic-partials-visualizer.git
cd harmonic-partials-visualizer

# Inicie um servidor local
python3 -m http.server 8000
# Acesse http://localhost:8000
```

Não há build steps, transpilação ou bundling. Edite `index.html` e recarregue o navegador.

## Código de conduta

- Seja respeitoso e construtivo
- Valorize contribuições de todos os níveis de experiência
- Priorize a clareza pedagógica — este é um projeto educacional

---

Dúvidas? Abra uma issue ou entre em contato com o mantenedor do projeto.
