# Construindo-um-Sistema-de-Vendas-com-Procedures-Functions-e-Triggers
## Grupo 2: Criação do Procedimento de Registro de Venda
### Composto por: Ana Lara, Sidney, Marquesi, Kauã Oliveira.
#### Código

DELIMITER //
CREATE PROCEDURE registrar_venda(
	IN id_produto INT,
	IN quantidade int,
	IN id_cliente INT,
	IN tipo_pagamento INT
)
 
BEGIN
	DECLARE total DECIMAL(10, 2);
	DECLARE estoque_suficiente BOOLEAN;
	DECLARE nova_quantidade_estoque INT;
	declare id_venda INT;
 
SET estoque_suficiente = verificar_estoque(id_produto, quantidade);
 
if NOT estoque_suficiente then
	SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Estoque insuficiente!';
END if;
 
SET total = calcular_total_venda(id_produto, quantidade);
 
INSERT INTO venda (id_produto, quantidade, id_cliente, data_venda)
VALUES (id_produto, quantidade, id_cliente, NOW());
 
SET id_venda = LAST_INSERT_ID();
 
INSERT INTO pagamento (id_venda, valor_pago, data_pagamento)
VALUES (id_venda, total, NOW());
 
SELECT quantidade_estoque INTO nova_quantidade_estoque
   FROM produtos
   WHERE id_produto = id_produto;
 
   UPDATE produtos
   SET quantidade_estoque = nova_quantidade_estoque - quantidade
   WHERE id_produto = id_produto;
 
   INSERT INTO estoque_histórico (id_produto, quantidade_alterada, data_alteracao)
   VALUES (id_produto, -quantidade, NOW());
END //
 
DELIMITER ;
