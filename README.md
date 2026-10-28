# cbro-clinica-case

Case de um sistema de gestão de clínica médica em produção: painel web da recepção, bot de WhatsApp com IA e app Android, todos usados no dia a dia da clínica.

> Este repositório é só um **case**, sem o código-fonte. O sistema lida com dados de saúde de pacientes (LGPD) e está em produção, então o código fica privado. Os detalhes técnicos podem ser mostrados numa conversa.

## O problema

A recepção agendava pelo WhatsApp e por papel, montava laudos e orçamentos à mão e conferia o faturamento do convênio em PDFs soltos. Tudo isso dependia de memória: lente de contato esquecida, exame sem preparo confirmado, guia perdida.

## O que o sistema faz

**Painel da recepção (web)**
- Agenda com agendas separadas, incluindo os sábados de exame.
- Histórico e prontuário do paciente.
- Documentos gerados a partir de modelos editáveis: relatório e justificativa de exame por CID, e orçamento de cirurgia em PDF.
- Fluxos de acompanhamento:
  - **cirurgias:** exames → autorização → agendada → realizada;
  - **lentes de contato:** solicitada → pedida → chegou → entregue.
- **Caixa e faturamento:** upload semanal dos PDFs do convênio. O sistema lê os PDFs e gera resumo, gráfico e um documento agregado, sem nomes de pacientes, para o médico.
- **WhatsApp dentro do painel:** a recepção lê as conversas e responde pelo site.
- **Perfil médico** com acesso restrito (agenda só leitura, histórico e prontuário), aplicado por lista de permissões no servidor.

**Bot de WhatsApp com IA**
- Atende pacientes, entende a intenção da mensagem e agenda. O menu fica só como rede de segurança.
- Memória da conversa entre dias, e trava que escala para a recepção quando necessário.
- Fica em silêncio quando um atendente humano responde.
- Confirmação automática de consultas e exames, com instruções de preparo.
- Leitura de guias por IA (OCR), de 45 s para 2 s por guia depois de otimizado.

**App Android**
- Acesso ao painel pelo celular, com login por biometria.
- App irmão para digitalizar guias: [guias-scanner-app](https://github.com/GABRIELFERR9/guias-scanner-app).

## Arquitetura

```
Paciente ── WhatsApp ── Evolution API ── Bot (Node.js + IA) ──┐
                                                              ├── PostgreSQL
Recepção / Médico ── Painel web (nginx + Node.js/Express) ────┘
                         └── App Android (Capacitor)
```

- **Backend:** Node.js e Express, com PostgreSQL (índices de busca), processos no PM2, nginx com gzip e timeouts, numa VPS Linux.
- **WhatsApp:** Evolution API, que o painel acessa por um proxy no servidor.
- **IA:** modelos de linguagem e de visão com cota controlada e fallback entre provedores.
- **Mobile:** Capacitor, com biometria nativa.

## O que aprendi

- **Diagnosticar pelo comportamento real, não pelo log.** Um buffer de stdout fez o bot parecer parado por horas, e ele estava funcionando.
- **IA em produção precisa de trava:** limite de cota, timeout curto e caminho de escape para um humano.
- **Dado sensível muda o desenho do sistema:** o documento para o médico sai agregado e sem nomes.

---
Feito por [Gabriel Ferreira](https://github.com/GABRIELFERR9).
