---
description: Cria vídeo de validação de User Story com legendas explicativas
allowed-tools: mcp__youtrack__*, mcp__gitlab__*, Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, Task
---

**IMPORTANTE:** Antes de executar, carregue a skill `gv-youtrack` para entender o workflow do YouTrack no Grupo Voalle.

# Validação de User Story

Cria um vídeo de validação E2E para demonstrar as funcionalidades implementadas em uma User Story.

## Argumentos

- `$ARGUMENTS` - ID da User Story no YouTrack (ex: ORN-1148)

## Processo

### 1. Buscar informações da US no YouTrack

```
mcp__youtrack__get_issue com issueId = $ARGUMENTS
```

Extrair:
- Summary (título)
- Description (funcionalidades a validar)
- Status atual
- Tasks relacionadas (linkedIssueCounts)

### 2. Buscar Merge Request relacionado

Procurar MR no GitLab que contenha o ID da US:

```
mcp__gitlab__list_merge_requests com search = $ARGUMENTS
```

Se encontrar MR:
- Obter diffs para entender o código implementado
- Identificar arquivos modificados
- Extrair informações relevantes para os cenários de teste

O código fonte é essencial para entender exatamente o que foi implementado e criar cenários de validação precisos.

### 3. Identificar skill de QA do projeto

Procurar skills de QA disponíveis no projeto:

```bash
find .claude/skills -name "*.md" -exec grep -l -i "qa\|test\|e2e\|playwright" {} \;
```

Se encontrar skill de QA (ex: `qa-omnichannel`, `qa-frontend`):
- Ler instruções da skill
- Usar configurações e padrões documentados
- Seguir estrutura de arquivos recomendada

Se NÃO encontrar skill de QA:
- Perguntar ao usuário usando AskUserQuestion:
  > Não encontrei uma skill de QA para este projeto. Como devo executar os testes E2E?
  > Opções: Playwright, Cypress, Outro (especificar)

### 4. Identificar ambiente de teste

Procurar configuração de ambiente em:
- `.env.local` ou `.env` (variáveis PLAYWRIGHT_*, CYPRESS_*, etc.)
- Arquivos de configuração do framework de teste
- Skill de QA (se existir)

Se NÃO encontrar ambiente configurado:
- Perguntar ao usuário usando AskUserQuestion:
  > Qual o host/URL do ambiente para executar a validação?
  > Exemplos: http://localhost:3000, https://staging.exemplo.com

### 5. Criar teste de validação

Criar arquivo de teste seguindo o padrão do projeto. Template base:

```typescript
/**
 * Validação da US {ID} - {Summary}
 *
 * Este teste grava um vídeo de validação com legendas explicativas
 * para demonstrar as funcionalidades implementadas.
 *
 * Funcionalidades testadas:
 * {Lista extraída da description e código do MR}
 *
 * Merge Request: !{MR_IID} - {MR_TITLE}
 */

// Helper para mostrar legenda na tela
async function showSubtitle(page: any, text: string, durationMs: number = 3000) {
  await page.evaluate((subtitle: string) => {
    const existing = document.getElementById('e2e-subtitle');
    if (existing) existing.remove();

    const div = document.createElement('div');
    div.id = 'e2e-subtitle';
    div.style.cssText = `
      position: fixed;
      bottom: 50px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 0, 0, 0.85);
      color: white;
      padding: 16px 32px;
      border-radius: 8px;
      font-size: 18px;
      font-family: system-ui, -apple-system, sans-serif;
      z-index: 999999;
      max-width: 80%;
      text-align: center;
      box-shadow: 0 4px 20px rgba(0,0,0,0.3);
      animation: fadeIn 0.3s ease-in;
    `;
    div.textContent = subtitle;

    const style = document.createElement('style');
    style.textContent = `
      @keyframes fadeIn {
        from { opacity: 0; transform: translateX(-50%) translateY(20px); }
        to { opacity: 1; transform: translateX(-50%) translateY(0); }
      }
    `;
    document.head.appendChild(style);
    document.body.appendChild(div);
  }, text);

  await page.waitForTimeout(durationMs);
}

async function hideSubtitle(page: any) {
  await page.evaluate(() => {
    const existing = document.getElementById('e2e-subtitle');
    if (existing) existing.remove();
  });
}

/**
 * Mostra a URL atual em um banner no topo da página (simula barra de endereço do navegador)
 * SEMPRE usar após cada page.goto() para mostrar a URL no vídeo
 */
async function showUrlBar(page: any, url: string) {
  await page.evaluate((currentUrl: string) => {
    const existing = document.getElementById('qa-url-bar');
    if (existing) existing.remove();

    const urlBar = document.createElement('div');
    urlBar.id = 'qa-url-bar';
    urlBar.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      background: #f1f3f4;
      border-bottom: 1px solid #ddd;
      padding: 8px 16px;
      font-family: system-ui, -apple-system, sans-serif;
      font-size: 14px;
      z-index: 999999;
      display: flex;
      align-items: center;
      gap: 12px;
    `;

    const lockIcon = currentUrl.startsWith('https') ? '🔒' : '⚠️';
    const urlObj = new URL(currentUrl);
    const domain = urlObj.host;
    const path = urlObj.pathname + urlObj.search;

    urlBar.innerHTML = `
      <span style="font-size: 16px;">${lockIcon}</span>
      <div style="
        flex: 1;
        background: white;
        border-radius: 20px;
        padding: 6px 16px;
        border: 1px solid #ddd;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      ">
        <span style="color: #202124; font-weight: 500;">${domain}</span><span style="color: #5f6368;">${path.length > 60 ? path.substring(0, 60) + '...' : path}</span>
      </div>
    `;

    document.body.appendChild(urlBar);
    document.body.style.paddingTop = '50px';
  }, url);
}

async function hideUrlBar(page: any) {
  await page.evaluate(() => {
    const urlBar = document.getElementById('qa-url-bar');
    if (urlBar) urlBar.remove();
    document.body.style.paddingTop = '';
  });
}

const PAUSE_DURATION = 3000;

// Adaptar imports e estrutura conforme skill de QA do projeto
test.describe('{ID} - Validação: {Summary}', () => {
  test('Vídeo de validação completo', async ({ page }) => {
    // INTRODUÇÃO (tela inicial sem URL bar)
    await page.goto('about:blank');
    await page.setContent(`
      <div style="display:flex;flex-direction:column;align-items:center;justify-content:center;height:100vh;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:white;font-family:system-ui;">
        <h1 style="font-size:48px;">US {ID}</h1>
        <h2 style="font-size:28px;font-weight:normal;opacity:0.9;">{Summary}</h2>
        <p style="font-size:18px;opacity:0.7;margin-top:32px;">Validação QA</p>
      </div>
    `);
    await page.waitForTimeout(4000);

    // ========================================
    // CENÁRIOS DE TESTE
    // IMPORTANTE: Após cada page.goto(), SEMPRE chamar:
    //   await showUrlBar(page, url);
    // Isso mostra a URL no topo da página (simula barra do navegador)
    //
    // Gerar cenários com base em:
    // 1. Description da US
    // 2. Código do Merge Request
    // 3. Arquivos modificados
    // ========================================

    // Exemplo de navegação com URL bar:
    // const testUrl = `${baseUrl}/minha-rota?param=valor`;
    // await page.goto(testUrl, { waitUntil: 'networkidle' });
    // await showUrlBar(page, testUrl);  // <-- SEMPRE incluir!
    // await showSubtitle(page, 'Verificando funcionalidade X...');

    // CONCLUSÃO (remover URL bar antes da tela final)
    await hideSubtitle(page);
    await hideUrlBar(page);
    await page.setContent(`
      <div style="display:flex;flex-direction:column;align-items:center;justify-content:center;height:100vh;background:linear-gradient(135deg,#11998e 0%,#38ef7d 100%);color:white;font-family:system-ui;text-align:center;">
        <h1 style="font-size:48px;">✅ Validação Concluída</h1>
        <div style="font-size:24px;line-height:1.8;">
          <p>✓ Funcionalidade 1 OK</p>
          <p>✓ Funcionalidade 2 OK</p>
        </div>
        <p style="font-size:16px;opacity:0.7;margin-top:48px;">US {ID} - Validado com sucesso</p>
      </div>
    `);
    await page.waitForTimeout(5000);
  });
});
```

### 6. Adaptar cenários de teste

Analisar a description da US E o código do MR para criar cenários precisos:
- Cada funcionalidade deve ter uma seção com legenda explicativa
- Usar pausas de 3 segundos entre ações
- Legendas devem explicar o que está sendo testado
- Usar URLs e dados reais quando possível
- Cobrir casos de sucesso e edge cases identificados no código

**IMPORTANTE - Barra de URL:**
- SEMPRE chamar `showUrlBar(page, url)` após cada `page.goto()`
- Isso mostra uma barra de endereço simulada no topo da página
- A URL fica visível no vídeo, facilitando a compreensão do QA
- Antes da tela de conclusão, chamar `hideUrlBar(page)` para limpar

### 7. Executar teste e gravar vídeo

Usar comando de teste conforme skill de QA ou configuração do projeto:

```bash
# Exemplo Playwright
npx playwright test validation-video-{id}.spec.ts --config={CONFIG_PATH} --headed

# Exemplo Cypress
npx cypress run --spec "cypress/e2e/validation-video-{id}.cy.ts"
```

### 8. Copiar vídeo para local acessível

Localizar vídeo gerado e copiar para local de fácil acesso:

```bash
# Ajustar caminho conforme framework
cp "{OUTPUT_PATH}/video.webm" "{PROJECT_PATH}/validation-video-{id}.webm"
```

### 9. Informar usuário sobre anexo

Informar que o MCP do YouTrack não suporta upload de arquivos e fornecer:
- Caminho do vídeo local
- Comando para abrir o vídeo
- Link do card no YouTrack para anexar manualmente

### 10. Perguntar sobre finalização

Perguntar ao usuário usando AskUserQuestion:
> Deseja que eu adicione tempo de trabalho e mova o card para Backlog-QA?

Se sim:
1. Perguntar duração do trabalho (sugestão: 30 minutos)
2. Registrar tempo de trabalho (workType: "Automation E2E" ou "Validation")
3. Mover status para "Backlog-QA"
4. Verificar se a mudança foi aplicada (seguir workflow do YouTrack)

## Checklist de Informações Necessárias

Antes de criar o teste, garantir que temos:

- [ ] ID da US no YouTrack
- [ ] Description/funcionalidades a validar
- [ ] Merge Request com código implementado
- [ ] Skill de QA ou framework de teste
- [ ] Ambiente/URL para executar teste
- [ ] Configuração de autenticação (se necessário)

## Exemplo de Uso

```
/validation ORN-1148
```

Fluxo:
1. Busca US ORN-1148 no YouTrack
2. Busca MR relacionado no GitLab
3. Identifica skill qa-omnichannel no projeto
4. Usa ambiente ia-staging.voalle.com.br (da config)
5. Cria teste com cenários baseados na US + código do MR
6. Grava vídeo com legendas explicativas
7. Fornece instruções para anexar no YouTrack
8. Opcionalmente adiciona tempo e move para Backlog-QA
