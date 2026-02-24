# 📦 Cubagem AR

Aplicação de Realidade Aumentada para simulação de empilhamento de caixas em paletes de armazém, desenvolvida inteiramente em HTML/JS puro com WebXR e Three.js.

---

## Visão Geral

O Cubagem AR permite posicionar um palete virtual no chão real do seu ambiente e empilhar caixas sobre ele seguindo regras logísticas de cores. A aplicação roda diretamente no navegador — sem instalação de app — e utiliza a câmera do dispositivo para ancorar os objetos 3D no mundo físico via WebXR Hit Test.

A cada sessão, caixas são geradas aleatoriamente em três categorias (vermelho, verde e azul), com dimensões e volumes proporcionais às suas regras de classe. O usuário posiciona, move e remove caixas em tempo real, enquanto um painel de estatísticas exibe o total empilhado e o percentual de preenchimento do palete.

---

## Funcionalidades

- **Posicionamento de palete por toque** — aponte para o chão e toque uma vez para ancorar o palete (1,2 m × 0,8 m) na superfície detectada.
- **Inserção de caixas** — fantasma translúcido mostra exatamente onde a caixa vai pousar antes de confirmar o toque.
- **Mover caixas** — selecione o modo mover e mova uma caixa; as caixas empilhadas acima da caixa selecionada irão cair no chão (ou sobre a caixa imediatamente abaixo) assim que a base for deslocada.
- **Remover caixas** — selecione o modo remover para excluir uma caixa; as caixas empilhadas acima da caixa removida cairão automaticamente para preencher o espaço vazio.
- **Regras de empilhamento por cor** — validação em tempo real com mensagem de erro exibida na tela.
- **Regra de suporte mínimo (50 %)** — só é permitido empilhar quando ao menos 70 % da base da caixa superior está apoiada sobre a caixa inferior; posicionamentos na borda são bloqueados.
- **Estatísticas em tempo real** — total de caixas, contagem por cor e percentual de preenchimento volumétrico do palete.
- **Reset e saída** — botões dedicados para limpar todas as caixas ou encerrar a sessão AR; ao reiniciar, o estado é completamente zerado.

---

## Controles

A interface AR é composta por três áreas:

### Barra superior
Exibe a fase atual (ex.: *"Posicionando caixas"*) e uma dica contextual à direita que muda conforme o modo ativo.

### Barra de modos (pill inferior)
Três botões independentes para alternar o modo de interação:

| Botão | Modo | Como usar |
|---|---|---|
| **＋ Inserir** | Inserção | O fantasma da próxima caixa aparece no palete. Toque para confirmar o posicionamento. |
| **⇄ Mover** | Movimentação | Toque em uma caixa para agarrá-la (o contorno fica laranja). Arraste apontando para o destino e toque novamente para soltar. |
| **🗑 Remover** | Remoção | Aponte para uma caixa (contorno vermelho). Mantenha o toque por **0,5 segundos** para removê-la. |

### Botões utilitários (canto superior esquerdo)

| Botão | Ação |
|---|---|
| **↺ Reset** | Remove todas as caixas e volta ao modo Inserir. O palete permanece. |
| **✕ Sair** | Encerra a sessão AR e retorna à tela inicial. |

### Card de próxima caixa (canto superior direito)
Visível apenas no modo Inserir. Mostra a cor, as dimensões (em cm) e o volume (em litros) da próxima caixa a ser inserida.

---

## Regras de Empilhamento

As caixas são geradas em três classes com volumes e restrições de empilhamento distintos:

| Cor | Classe | Volume | Pode ser empilhada sobre |
|---|---|---|---|
| 🔴 **Vermelho** | Grande | > 27 L | Superfície do palete ou outra caixa **vermelha** |
| 🟢 **Verde** | Médio | 8 – 27 L | Caixas **vermelhas** ou **verdes** |
| 🔵 **Azul** | Pequeno | < 8 L | Qualquer cor |

**Regra de suporte:** independentemente da cor, uma caixa só pode ser posicionada sobre outra se ao menos **50 % de sua base** estiver apoiada. Tentar empilhar na borda exibe o aviso *"Apoio insuficiente — mova para o centro"*.

Ao tentar violar uma regra de cor, um ícone **✕** vermelho é exibido no centro da tela junto com a mensagem de motivo (ex.: *"Vermelho só pode ser empilhado em vermelho."*).

---

## Configurações do Projeto

Todos os parâmetros ajustáveis estão concentrados no bloco `CONSTANTS` no início do `<script type="module">` do arquivo `index.html`:

```js
// Dimensões do palete (em metros)
const PAL_W   = 1.20;   // largura
const PAL_D   = 0.80;   // profundidade
const PAL_H   = 0.14;   // altura

// Tempo de toque para deletar uma caixa (em milissegundos)
const HOLD_MS = 500;
```

### Tamanhos e volumes das caixas

Na função `randomBox()`, logo abaixo dos constants:

```js
// Volumes por classe (em m³)
// Vermelho  — entre 28 L e 60 L
vol = 0.028 + Math.random() * 0.032;

// Verde     — entre 8 L e 27 L
vol = 0.008 + Math.random() * 0.019;

// Azul      — entre 1 L e 8 L
vol = 0.001 + Math.random() * 0.007;

// Altura aleatória (em metros)
const h = 0.06 + Math.random() * 0.22;   // entre 6 cm e 28 cm

// Limites de largura e profundidade (em metros)
const w = Math.max(0.06, Math.min(0.38, ...));  // entre 6 cm e 38 cm
const d = Math.max(0.06, Math.min(0.38, ...));  // entre 6 cm e 38 cm
```

### Suporte mínimo para empilhamento

Na função `hasEnoughSupport()`:

```js
return (overlapX * overlapZ) >= 0.50 * top.w * top.d;
// Altere 0.50 para ajustar o percentual mínimo de base apoiada (0.0 a 1.0)
```

### Distribuição de cores

Na função `randomBox()`, a distribuição padrão é **1/3 para cada cor**:

```js
if (r < 0.333)       cls = 'red';
else if (r < 0.667)  cls = 'green';
else                 cls = 'blue';
// Ajuste os limiares para favorecer uma cor específica
```

### Suavização do hit-test (estabilidade do cursor no chão)

```js
const ALPHA = 0.25;   // fator de suavização XZ (0 = imóvel, 1 = sem suavização)
// O Y do chão usa o percentil 10 dos últimos 30 leituras para estabilidade
stableY = s[Math.floor(s.length * 0.10)];
```

---

## Tecnologias Utilizadas

| Tecnologia | Versão | Uso |
|---|---|---|
| **WebXR Device API** | — | Sessão AR imersiva, hit-test no chão, eventos de toque (selectend) |
| **WebXR Hit Test API** | — | Detecção de superfícies reais para posicionamento do palete |
| **Three.js** | r128 | Renderização 3D (caixas, palete, fantasma, contornos, sprites de label) |
| **HTML / CSS / JS puro** | ES2020+ | Interface do usuário (overlay AR, pill de modos, cards, estatísticas) |

### Requisitos de dispositivo

- **Sistema operacional:** Android
- **Navegador:** Google Chrome (versão 81+)
- **Hardware:** dispositivo com suporte a **ARCore** (Google Play Services for AR)
- **Protocolo:** a página deve ser servida via **HTTPS** ou `localhost` (requisito do WebXR)

### Como servir localmente

```bash
# Python 3
python -m http.server 8080

# Node.js (npx)
npx serve .
```

Acesse `https://<ip-local>:8080/logistics-ar.html` no Chrome do celular. Para HTTPS local, ferramentas como [ngrok](https://ngrok.com) ou [mkcert](https://github.com/FiloSottile/mkcert) podem ser usadas.

---

## Estrutura do Arquivo

O projeto é um **único arquivo HTML autocontido** (`logistics-ar.html`):

```
index.html
```