# Faixa Viva

Protótipo de monitoramento inteligente da faixa de domínio — proposta para o Desafio de Inovação Motiva.

## Conteúdo

- `index.html` — protótipo interativo (dashboard). Arquivo único, autocontido (sem dependências externas), pronto para hospedagem estática.
- `faixa_viva_proposta.docx` — documento de proposta técnica que acompanha o protótipo.

## Rodar localmente

Basta abrir `index.html` em qualquer navegador. Não precisa de servidor, build ou instalação.

## Publicar

Como é um único arquivo estático, pode ser hospedado em qualquer serviço gratuito:

- **GitHub Pages**: em Settings → Pages deste repositório, ative o Pages a partir da branch `main` (pasta raiz). O site fica em `https://<usuario>.github.io/faixa-viva/`. Depois, em Settings → Pages → "Custom domain", pode-se apontar um domínio próprio (ex.: faixaviva.com.br), configurando o DNS conforme a documentação do GitHub Pages.
- **Netlify / Vercel**: arrastar o `index.html` na interface de deploy manual.

## Contexto

Dados de vegetação, custos de roçada e séries temporais no protótipo são majoritariamente simulados (com semente fixa, para reprodutibilidade), com exceção dos custos de referência do SICRO/DNIT citados diretamente no conteúdo. Ver a seção "Como isso funcionaria com dados reais" dentro do próprio protótipo para o plano de uso de dados reais.
