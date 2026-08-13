# Faixa Viva

Protótipo de monitoramento inteligente da faixa de domínio — proposta para o Desafio de Inovação Motiva.

## Conteúdo

- `index.html` — protótipo interativo (dashboard). Sem build, sem dependências externas em runtime (nenhuma chamada de API acontece no navegador do usuário).
- `images/` — 4 imagens reais de satélite (Sentinel-2), uma por rodovia/trecho, buscadas uma única vez via Copernicus Data Space Ecosystem e servidas como arquivo estático.
- `faixa_viva_proposta.docx` — documento de proposta técnica que acompanha o protótipo.

## Rodar localmente

Basta abrir `index.html` em qualquer navegador (as imagens em `images/` carregam por caminho relativo, então mantenha a pasta junto). Não precisa de servidor, build ou instalação.

## Publicar

Hospede a pasta inteira (não só o `index.html`) em qualquer serviço estático:

- **GitHub Pages**: em Settings → Pages deste repositório, ative o Pages a partir da branch `main` (pasta raiz). O site fica em `https://<usuario>.github.io/faixa-viva/`. Depois, em Settings → Pages → "Custom domain", pode-se apontar um domínio próprio (ex.: faixaviva.com.br), configurando o DNS conforme a documentação do GitHub Pages.
- **Netlify / Vercel**: conectar o repositório (ou arrastar a pasta inteira, não só o `index.html`, na interface de deploy manual).

## Contexto

Dados de vegetação, custos de roçada e séries temporais no protótipo são majoritariamente simulados (com semente fixa, para reprodutibilidade), com exceção dos custos de referência do SICRO/DNIT citados diretamente no conteúdo. Ver a seção "Como isso funcionaria com dados reais" dentro do próprio protótipo para o plano de uso de dados reais.

Na rodovia Fernão Dias, a geometria dos trechos (posições em km e coordenadas) é real, extraída da relação `BR-381` no OpenStreetMap via Overpass API — não simulada. Cada trecho tem um link "Ver coordenada real no mapa" que abre a localização real no OpenStreetMap. A altura de vegetação em acostamentos e as causas de alerta continuam simuladas; a extensão real do trecho inicial mapeado (São Paulo/SP até a divisa com Minas Gerais) é de ~87 km. Autoban e Motiva Paraná ainda usam geometria de trecho totalmente simulada.

A cobertura vegetal dos 5 trechos de talude da Fernão Dias é real: NDVI do Sentinel-2, obtido via Copernicus Data Space Ecosystem (Statistical API) para as coordenadas reais de cada trecho — busca única, gravada estática no arquivo (nenhuma chamada de API acontece em produção). O limiar de escalada para alerta dessa métrica foi recalibrado (ver comentário no código, seção `computeSegments`), porque o limiar de 50% calibrado para as curvas sintéticas teria sinalizado quase todo trecho real como alerta — cobertura vegetal real de talude fica tipicamente entre 40–70%.

A seção "Evidência real" traz 4 imagens de satélite reais (Sentinel-2, RGB verdadeiro), uma de cada uma das 3 rodovias operadas pela Motiva usadas no protótipo (Fernão Dias, Autoban, Motiva Paraná/BR-376), buscadas via Copernicus para as coordenadas reais de cada trecho. Cada card tem um toggle "Foto real" / "Índice de vegetação" que troca para uma segunda imagem real — a mesma cena renderizada por NDVI (verde = cobertura densa, marrom = esparsa/solo exposto) — acompanhada do valor real de NDVI (%) e a data da leitura. Não são câmeras ao vivo — a Fernão Dias ainda não tem câmeras instaladas, e o Sentinel-2 revisita cada ponto a cada ~5 dias, não em tempo real.

Para regenerar essas imagens/NDVI com dados mais recentes, é necessário um Client ID/Secret gratuito da Copernicus Data Space Ecosystem (ver `.env.local`, fora do git) — as chamadas ficam em scripts Python usados uma única vez, nunca embutidas no `index.html`.
