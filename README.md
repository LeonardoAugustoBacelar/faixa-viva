# Faixa Viva

Protótipo de monitoramento inteligente da faixa de domínio — proposta para o Desafio de Inovação Motiva.

## Conteúdo

- `index.html` — protótipo interativo (dashboard). Sem build, sem dependências externas em runtime (nenhuma chamada de API acontece no navegador do usuário).
- `sentinel-status.html` — página dedicada de status técnico do monitoramento por satélite (cobertura NDVI + radar, specs dos sensores, matriz de 90 dias por trecho). Acessível pelo link na seção "Evidência real" do dashboard.
- `mapa.html` — mapa ao vivo em tela cheia (estilo Windy.com/NASA Worldview) com as rotas reais das 3 rodovias e os 7 pontos monitorados, mais uma linha do tempo arrastável pelas 6 datas reais verificadas. Única página do projeto com dependência externa em runtime: carrega o Leaflet (JS/CSS, via CDN unpkg.com) e os tiles do basemap escuro da CARTO/OpenStreetMap — precisa de internet para funcionar, diferente do resto do site.
- `images/` — 8 imagens reais de satélite (Sentinel-2), foto real + índice de vegetação por rodovia/trecho, buscadas uma única vez via Copernicus Data Space Ecosystem e servidas como arquivo estático.
- `faixa_viva_proposta.docx` — documento de proposta técnica que acompanha o protótipo.

## Rodar localmente

Basta abrir `index.html` em qualquer navegador (as imagens em `images/` carregam por caminho relativo, então mantenha a pasta junto). Não precisa de servidor, build ou instalação.

## Publicar

Hospede a pasta inteira (não só o `index.html`) em qualquer serviço estático:

- **GitHub Pages**: em Settings → Pages deste repositório, ative o Pages a partir da branch `main` (pasta raiz). O site fica em `https://<usuario>.github.io/faixa-viva/`. Depois, em Settings → Pages → "Custom domain", pode-se apontar um domínio próprio (ex.: faixaviva.com.br), configurando o DNS conforme a documentação do GitHub Pages.
- **Netlify / Vercel**: conectar o repositório (ou arrastar a pasta inteira, não só o `index.html`, na interface de deploy manual).

## Contexto

Dados de vegetação, custos de roçada e séries temporais no protótipo são majoritariamente simulados (com semente fixa, para reprodutibilidade), com exceção dos custos de referência do SICRO/DNIT citados diretamente no conteúdo.

Nas 3 rodovias, a geometria dos trechos (posições em km e coordenadas) é real, extraída da relação de cada rodovia no OpenStreetMap via Overpass API — não simulada. Cada trecho tem um link "Ver coordenada real no mapa" que abre a localização real no OpenStreetMap. Fernão Dias: ~87 km reais (São Paulo/SP até a divisa com Minas Gerais). Autoban: ~58,4 km reais (Rodovia dos Bandeirantes rumo a Campinas). Motiva Paraná: ~14,6 km reais (descida da Serra do Mar, BR-376, rumo a Paranaguá). A altura de vegetação em acostamentos e as causas de alerta continuam simuladas em todas as três.

A cobertura vegetal dos 5 trechos de talude da Fernão Dias é real: NDVI do Sentinel-2, obtido via Copernicus Data Space Ecosystem (Statistical API) para as coordenadas reais de cada trecho — busca única, gravada estática no arquivo (nenhuma chamada de API acontece em produção). O limiar de escalada para alerta dessa métrica foi recalibrado (ver comentário no código, seção `computeSegments`), porque o limiar de 50% calibrado para as curvas sintéticas teria sinalizado quase todo trecho real como alerta — cobertura vegetal real de talude fica tipicamente entre 40–70%.

A seção "Evidência real" traz 4 imagens de satélite reais (Sentinel-2, RGB verdadeiro), uma de cada uma das 3 rodovias operadas pela Motiva usadas no protótipo (Fernão Dias, Autoban, Motiva Paraná/BR-376), buscadas via Copernicus para as coordenadas reais de cada trecho. Cada card tem um toggle "Foto real" / "Índice de vegetação" que troca para uma segunda imagem real — a mesma cena renderizada por NDVI (verde = cobertura densa, marrom = esparsa/solo exposto) — acompanhada do valor real de NDVI (%) e a data da leitura. Não são câmeras ao vivo — a Fernão Dias ainda não tem câmeras instaladas, e o Sentinel-2 revisita cada ponto a cada ~5 dias, não em tempo real.

Cada um dos 5 trechos de talude com NDVI real também tem uma camada de radar (Sentinel-1 GRD, banda VV) buscada nas mesmas janelas de 15 dias — visível no painel de detalhe como "Cobertura de monitoramento", e detalhada por completo em `sentinel-status.html`. O radar atravessa nuvem e funciona de dia ou de noite: no período verificado, o óptico teve leitura válida em 20 das 30 janelas possíveis (67%, por causa de nuvem) e o radar cobriu as 30 (100%). O radar mede refletividade de superfície, não é um índice de vegetação — não substitui o NDVI, só garante que nenhum trecho fica sem leitura no período.

Para regenerar essas imagens/NDVI/radar com dados mais recentes, é necessário um Client ID/Secret gratuito da Copernicus Data Space Ecosystem (ver `.env.local`, fora do git) — as chamadas ficam em scripts Python usados uma única vez, nunca embutidas no `index.html`.

Dois recursos do painel de detalhe rodam sobre esses mesmos dados reais, sem API nova:

- **Projeção real de tendência** — para os 5 trechos com NDVI real, uma regressão linear simples sobre os pontos reais estima a velocidade de mudança (p.p./semana) e, quando a trajetória aponta para a referência de 50%, projeta em quantos dias cruzaria — deixado explícito como extrapolação simples, não um modelo validado.
- **Chuva real ao vivo** — única chamada de API que roda de verdade no navegador do usuário (as demais são buscas únicas, gravadas estáticas). Usa a API gratuita e sem chave do Open-Meteo (CORS liberado) para buscar precipitação real dos últimos 7 dias e a previsão real dos próximos 7, para a coordenada real de cada trecho — funciona nas 3 rodovias, já que todas têm geometria real. Sinaliza risco elevado quando um talude com causa de erosão tem ≥20mm previstos.
- **OS automática por chuva real** — fecha o ciclo entre os dois recursos acima e o fluxo de ordem de serviço: se a previsão real de chuva (Open-Meteo) para um talude com erosão já sinalizada passa de 20mm/7 dias e nenhuma OS foi emitida ainda, o sistema emite uma sozinho, sem clique manual. Aparece com selo distinto ("emitida automaticamente", cor diferente da OS manual) no painel de detalhe e no app de campo, e o texto deixa explícito o que é real (a chuva) e o que é simulado (o disparo em si — em produção, integraria com o sistema de manutenção da concessionária).
