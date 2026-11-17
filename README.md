--- RESPOSTAS DAS PERGUNTAS HOTEL NEBULA ---

1 - quais quartos estão disponíveis em determinado período?
R: 202, 301 e 401

-- comando SQL que utilizei para resolver: 
USE hotel_nebula;

SELECT 
    q.id_quarto,
    q.numero_quarto,
    q.tipo_quarto,
    q.capacidade_hospedes,
    q.valor_diaria
FROM quartos q
WHERE q.status_quarto <> 'Manutencao'
  AND q.id_quarto NOT IN (
      SELECT r.id_quarto
      FROM reservas r
      WHERE r.status_reserva IN ('Pendente', 'Confirmada')
        AND r.data_checkin_previsto < '2026-11-25'
        AND r.data_checkout_previsto > '2026-11-20'
  )
  AND q.id_quarto NOT IN (
      SELECT h.id_quarto
      FROM hospedagens h
      WHERE h.status_hospedagem = 'Ativa'
  )
ORDER BY q.numero_quarto;

2. Quais hóspedes mais realizaram reservas?
Mariana Costa Almeida

SELECT 
    h.id_hospede,
    h.nome_completo,
    h.email,
    COUNT(r.id_reserva) AS total_reservas
FROM hospedes h
LEFT JOIN reservas r ON h.id_hospede = r.id_hospede
GROUP BY h.id_hospede, h.nome_completo, h.email
ORDER BY total_reservas DESC, h.nome_completo ASC;

3. Qual foi o faturamento por mês?
R: os faturamentos mensais foram 2026-09 (R$ 3.090,00), 2026-10 (R$ 5.860,00) e 2026-11 (R$ 4.605,00


SELECT 
    DATE_FORMAT(data_pagamento, '%Y-%m') AS mes_referencia,
    COUNT(id_pagamento) AS qtd_pagamentos,
    SUM(valor_pago) AS faturamento_total
FROM pagamentos
WHERE status_pagamento = 'Aprovado'
GROUP BY DATE_FORMAT(data_pagamento, '%Y-%m')
ORDER BY mes_referencia ASC;

4. Quais serviços extras são mais consumidos?
   R: Café da Manhã no Quarto lidera com 10 unidades.

   SELECT 
    s.id_servico,
    s.nome_servico,
    SUM(hs.quantidade) AS total_unidades_consumidas,
    SUM(hs.quantidade * hs.valor_unitario_cobrado) AS receita_gerada
FROM servicos s
INNER JOIN hospedagem_servicos hs ON s.id_servico = hs.id_servico
GROUP BY s.id_servico, s.nome_servico
ORDER BY total_unidades_consumidas DESC, receita_gerada DESC;

5-Quais quartos receberam melhores avaliações?
R:Os quartos 101, 301 e 401 empatam no topo com média 5.00.


SELECT 
    q.id_quarto,
    q.numero_quarto,
    q.tipo_quarto,
    ROUND(AVG(a.nota), 2) AS media_avaliacao,
    COUNT(a.id_avaliacao) AS total_avaliacoes
FROM quartos q
INNER JOIN hospedagens hosp ON q.id_quarto = hosp.id_quarto
INNER JOIN avaliacoes a ON hosp.id_hospedagem = a.id_hospedagem
GROUP BY q.id_quarto, q.numero_quarto, q.tipo_quarto
ORDER BY media_avaliacao DESC, total_avaliacoes DESC;

6-Quais reservas foram canceladas?
R:2 reservas canceladas (de Beatriz Nogueira Santos e Camila Rocha Teixeira).

SELECT 
    r.id_reserva,
    h.nome_completo AS hospede,
    q.numero_quarto,
    q.tipo_quarto,
    r.data_reserva,
    r.data_checkin_previsto,
    r.data_checkout_previsto,
    r.observacoes
FROM reservas r
INNER JOIN hospedes h ON r.id_hospede = h.id_hospede
INNER JOIN quartos q ON r.id_quarto = q.id_quarto
WHERE r.status_reserva = 'Cancelada'
ORDER BY r.data_checkin_previsto DESC;

7. Quantas hospedagens cada profissional atendeu?
    R:Nos dados de teste, Fernanda Lima Souza aparece em primeiro com 3 atendimentos, e o gerente Roberto Gomes Vasconcelos aparece com 0.

   SELECT 
    p.id_profissional,
    p.nome_completo,
    p.cargo,
    COUNT(hosp.id_hospedagem) AS total_hospedagens_atendidas
FROM profissionais p
LEFT JOIN hospedagens hosp ON p.id_profissional = hosp.id_profissional
GROUP BY p.id_profissional, p.nome_completo, p.cargo
ORDER BY total_hospedagens_atendidas DESC, p.nome_completo ASC;
