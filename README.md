# case_stone

[STONE.pptx](https://github.com/buenoleonardo1/case_stone/files/6352737/STONE.pptx)

# INÍCIO

Importei a tabela em CSV para o MYSQL. Nomeei de "rentabilidadeanalíticab"

# CRIANDO TABELA CONTANDO AS TRANSAÇÕES POR CLIENTE/DATA

create table quantidade_transacoes_por_cidade (
select distinct Número_do_Cliente, Canal, Cidade, Data_Apuração, Quantidade_de_Transações, count(Quantidade_de_Transações) as Transacoes from rentabilidadeanalíticab
where Quantidade_de_Transações <> 0
group by Número_do_Cliente, Data_Apuração, Cidade);

# CLUSTERIZAÇÃO CIDADES: CRIANDO TABELA CIDADES

create table cidades (
select Cidade, count(Quantidade_de_Transações) as qtdd_transacoes, sum(Quantidade_de_Transações) as soma_transacoes, 
sum(Total) as total_receitas, sum(Custos_Total) as total_custos, sum(Margem_final) as margem_final, 
sum(Margem_final)/sum(Quantidade_de_Transações) as margem_por_qtdd_transacoes from rentabilidadeanalíticab
group by Cidade); 

# Com volume transacionado
select * from cidades1;
drop table cidades1;
create table cidades1 (
select Cidade, Volume_Transacionado, count(Quantidade_de_Transações) as qtdd_transacoes, sum(Quantidade_de_Transações) as soma_transacoes, 
sum(Total) as total_receitas, sum(Custos_Total) as total_custos, sum(Margem_final) as margem_final, 
sum(Margem_final)/sum(Quantidade_de_Transações) as margem_por_qtdd_transacoes from rentabilidadeanalíticab
where Volume_Transacionado <> 0
group by Cidade); 

# CLUSTERIZAÇÃO CIDADES: CRIAÇÃO DE TABELAS PARA CADA MÊS

# Outubro
create table outubro (select Número_do_Cliente as ID_outubro, Cidade as Cidade_outubro, count(Transacoes) as Transacoes_outubro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-10-01 00:00:00'
group by Número_do_Cliente, Cidade);

# Novembro
create table novembro (select Número_do_Cliente as ID_novembro, Cidade as Cidade_novembro, count(Transacoes) as Transacoes_novembro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-11-01 00:00:00'
group by Número_do_Cliente, Cidade);

# Dezembro
create table dezembro (select Número_do_Cliente as ID_dezembro, Cidade as Cidade_dezembro, count(Transacoes) as Transacoes_dezembro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-12-01 00:00:00'
group by Número_do_Cliente, Cidade);

# Janeiro
create table janeiro (select Número_do_Cliente as ID_janeiro, Cidade as Cidade_janeiro, count(Transacoes) as Transacoes_janeiro from quantidade_transacoes_por_cidade
where Data_Apuração = '2017-01-01 00:00:00'
group by Número_do_Cliente, Cidade);

# Fevereiro
create table fevereiro (select Número_do_Cliente as ID_fevereiro, Cidade as Cidade_fevereiro, count(Transacoes) as Transacoes_fevereiro from quantidade_transacoes_por_cidade
where Data_Apuração = '2017-02-01 00:00:00'
group by Número_do_Cliente, Cidade);

# CLUSTERIZAÇÃO CIDADES: CRIANDO TABELAS PARA MEDIR O CHURN E TEMPO DE VIDA POR MÊS

# Churn Outubro
create table outubro_novembro
(select * from outubro
left join novembro
on outubro.ID_outubro = novembro.ID_novembro);

create table saldo_out_nov
(select Cidade_outubro, count(Cidade_outubro)-count(Cidade_novembro) as Saldo, count(Cidade_outubro) as Ativos_out from outubro_novembro
group by Cidade_outubro);

create table churn_out_nov
(select Cidade_outubro, Saldo, Ativos_out, Saldo/Ativos_out as churn from saldo_out_nov
order by Cidade_outubro);

create table tempo_de_vida_out_nov
(select Cidade_outubro, churn as churn_outubro, 1/churn as tempo_de_vida_out from churn_out_nov
group by Cidade_outubro);

# Churn Novembro
create table novembro_dezembro
(select * from novembro
left join dezembro
on novembro.ID_novembro = dezembro.ID_dezembro);

create table saldo_nov_dez
(select Cidade_novembro, count(Cidade_novembro)-count(Cidade_dezembro) as Saldo, count(Cidade_novembro) as Ativos_nov from novembro_dezembro
group by Cidade_novembro);

create table churn_nov_dez
(select Cidade_novembro, Saldo, Ativos_nov, Saldo/Ativos_nov as churn from saldo_nov_dez
order by Cidade_novembro);

create table tempo_de_vida_nov_dez
(select Cidade_novembro, churn as churn_novembro, 1/churn as tempo_de_vida_nov from churn_nov_dez
group by Cidade_novembro);

# Churn Dezembro
create table dezembro_janeiro
(select * from dezembro
left join janeiro
on dezembro.ID_dezembro = janeiro.ID_janeiro);

create table saldo_dez_jan
(select Cidade_dezembro, count(Cidade_dezembro)-count(Cidade_janeiro) as Saldo, count(Cidade_dezembro) as Ativos_dez from dezembro_janeiro
group by Cidade_dezembro);

create table churn_dez_jan
(select Cidade_dezembro, Saldo, Ativos_dez, Saldo/Ativos_dez as churn from saldo_dez_jan
order by Cidade_dezembro);

create table tempo_de_vida_dez_jan
(select Cidade_dezembro, churn as churn_dezembro, 1/churn as tempo_de_vida_dez from churn_dez_jan
group by Cidade_dezembro);

# Churn Janeiro
create table janeiro_fevereiro
(select * from janeiro
left join fevereiro
on janeiro.ID_janeiro = fevereiro.ID_fevereiro);

create table saldo_jan_fev
(select Cidade_janeiro, count(Cidade_janeiro)-count(Cidade_fevereiro) as Saldo, count(Cidade_janeiro) as Ativos_jan from janeiro_fevereiro
group by Cidade_janeiro);

create table churn_jan_fev
(select Cidade_janeiro, Saldo, Ativos_jan, Saldo/Ativos_jan as churn from saldo_jan_fev
order by Cidade_janeiro);

create table tempo_de_vida_jan_fev
(select Cidade_janeiro, churn as churn_janeiro, 1/churn as tempo_de_vida_jan from churn_jan_fev
group by Cidade_janeiro);

# Tabela de churn e tempo de vida médio de todo período
create table tempo_de_vida_out_nov_dez
(select Cidade_outubro, churn_outubro, tempo_de_vida_out, churn_novembro, tempo_de_vida_nov from tempo_de_vida_out_nov
join tempo_de_vida_nov_dez
on tempo_de_vida_out_nov.Cidade_outubro = tempo_de_vida_nov_dez.Cidade_novembro);

create table tempo_de_vida_dez_jan_fev
(select Cidade_dezembro, churn_dezembro, tempo_de_vida_dez, churn_janeiro, tempo_de_vida_jan from tempo_de_vida_dez_jan
join tempo_de_vida_jan_fev
on tempo_de_vida_dez_jan.Cidade_dezembro = tempo_de_vida_jan_fev.Cidade_janeiro);

create table tempo_de_vida_all
(select Cidade_outubro as Cidade_churn, churn_outubro, tempo_de_vida_out, churn_novembro, tempo_de_vida_nov,
churn_dezembro, tempo_de_vida_dez, churn_janeiro, tempo_de_vida_jan from tempo_de_vida_out_nov_dez
join tempo_de_vida_dez_jan_fev
on tempo_de_vida_out_nov_dez.Cidade_outubro = tempo_de_vida_dez_jan_fev.Cidade_dezembro);

create table churn_media
(select Cidade_churn, (churn_outubro+churn_novembro+churn_dezembro+churn_janeiro)/4 as churn_media 
from tempo_de_vida_all);

# CLUSTERIZAÇÃO CIDADES: CHURN E TEMPO DE VIDA MEDIO

create table churn_e_tempo_de_vida_medio
(select Cidade_churn, churn_media, 1/churn_media as tempo_de_vida_medio from churn_media);

# CLUSTERIZAÇÃO CIDADES: TABELA DAS RECEITAS, CUSTOS, LUCRO, CHURN E E TEMPO DE VIDA MÉDIO

create table clusterizacao_cidade
(select Cidade, total_receitas as receita, total_custos as custos, margem_final as lucro, 
churn_media as churn, tempo_de_vida_medio as tempo_de_vida from cidades
join churn_e_tempo_de_vida_medio
on cidades.Cidade = churn_e_tempo_de_vida_medio.Cidade_churn);

# CLUSTERIZAÇÃO CIDADES: CIDADES MAIS LUCRATIVAS

create table Cidades_Lucrativas
(select Cidade, Lucro from clusterizacao_cidade
where Lucro > 0
order by Lucro desc);

# CLUSTERIZAÇÃO CIDADES: CIDADES SEM CHURN (não tiveram perda de clientes)

create table Cidades_sem_Churn
(select Cidade, Churn, tempo_de_vida from clusterizacao_cidade
where churn = 0
order by Cidade);

# CLUSTERIZAÇÃO CIDADES: CIDADES COM OS MELHORES TEMPOS DE VIDA MÉDIO (menores churns <> 0)

create table Cidades_melhores_tempos_de_vida
(select Cidade, Churn, tempo_de_vida from clusterizacao_cidade
where tempo_de_vida > 10
order by Churn);

# CLUSTERIZAÇÃO CANAIS: CRIANDO TABELA CANAIS

create table canais (
select Canal, count(Quantidade_de_Transações) as qtdd_transacoes, sum(Quantidade_de_Transações) as soma_transacoes, 
sum(Total) as total_receitas, sum(Custos_Total) as total_custos, sum(Margem_final) as margem_final, 
sum(Margem_final)/sum(Quantidade_de_Transações) as margem_por_qtdd_transacoes from rentabilidadeanalíticab
group by canal
order by canal); 

# CLUSTERIZAÇÃO CANAIS: CRIAÇÃO DE TABELAS PARA CADA MÊS

create table canais_outubro (select Número_do_Cliente as ID_outubro, Canal as Canal_outubro, count(Transacoes) as Transacoes_outubro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-10-01 00:00:00'
group by Número_do_Cliente, Canal);

create table canais_novembro (select Número_do_Cliente as ID_novembro, Canal as Canal_novembro, count(Transacoes) as Transacoes_novembro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-11-01 00:00:00'
group by Número_do_Cliente, Canal);

create table canais_dezembro (select Número_do_Cliente as ID_dezembro, Canal as Canal_dezembro, count(Transacoes) as Transacoes_dezembro from quantidade_transacoes_por_cidade
where Data_Apuração = '2016-12-01 00:00:00'
group by Número_do_Cliente, Canal);

create table canais_janeiro (select Número_do_Cliente as ID_janeiro, Canal as Canal_janeiro, count(Transacoes) as Transacoes_janeiro from quantidade_transacoes_por_cidade
where Data_Apuração = '2017-01-01 00:00:00'
group by Número_do_Cliente, Canal);

create table canais_fevereiro (select Número_do_Cliente as ID_fevereiro, Canal as Canal_fevereiro, count(Transacoes) as Transacoes_fevereiro from quantidade_transacoes_por_cidade
where Data_Apuração = '2017-02-01 00:00:00'
group by Número_do_Cliente, Canal);

# CLUSTERIZAÇÃO CANAIS: CRIANDO TABELAS PARA MEDIR O CHURN E TEMPO DE VIDA POR MÊS

# Churn Outubro
create table canais_outubro_novembro
(select * from canais_outubro
left join canais_novembro
on canais_outubro.ID_outubro = canais_novembro.ID_novembro);

create table canais_saldo_out_nov
(select Canal_outubro, count(Canal_outubro)-count(Canal_novembro) as Saldo, count(Canal_outubro) as Ativos_out from canais_outubro_novembro
group by Canal_outubro);

create table canais_churn_out_nov
(select Canal_outubro, Saldo, Ativos_out, Saldo/Ativos_out as churn from canais_saldo_out_nov
order by Canal_outubro);

create table canais_tempo_de_vida_out_nov
(select Canal_outubro, churn as churn_outubro, 1/churn as tempo_de_vida_out from canais_churn_out_nov
group by Canal_outubro);

# Churn Novembro
create table canais_novembro_dezembro
(select * from canais_novembro
left join canais_dezembro
on canais_novembro.ID_novembro = canais_dezembro.ID_dezembro);

create table canais_saldo_nov_dez
(select Canal_novembro, count(Canal_novembro)-count(Canal_dezembro) as Saldo, count(Canal_novembro) as Ativos_nov from canais_novembro_dezembro
group by Canal_novembro);

create table canais_churn_nov_dez
(select Canal_novembro, Saldo, Ativos_nov, Saldo/Ativos_nov as churn from canais_saldo_nov_dez
order by Canal_novembro);

create table canais_tempo_de_vida_nov_dez
(select Canal_novembro, churn as churn_novembro, 1/churn as tempo_de_vida_nov from canais_churn_nov_dez
group by Canal_novembro);

# Churn Dezembro
create table canais_dezembro_janeiro
(select * from canais_dezembro
left join canais_janeiro
on canais_dezembro.ID_dezembro = canais_janeiro.ID_janeiro);

create table canais_saldo_dez_jan
(select Canal_dezembro, count(Canal_dezembro)-count(Canal_janeiro) as Saldo, count(Canal_dezembro) as Ativos_dez from canais_dezembro_janeiro
group by Canal_dezembro);

create table canais_churn_dez_jan
(select Canal_dezembro, Saldo, Ativos_dez, Saldo/Ativos_dez as churn from canais_saldo_dez_jan
order by Canal_dezembro);

create table canais_tempo_de_vida_dez_jan
(select Canal_dezembro, churn as churn_dezembro, 1/churn as tempo_de_vida_dez from canais_churn_dez_jan
group by Canal_dezembro);

# Churn Janeiro
create table canais_janeiro_fevereiro
(select * from canais_janeiro
left join canais_fevereiro
on canais_janeiro.ID_janeiro = canais_fevereiro.ID_fevereiro);

create table canais_saldo_jan_fev
(select Canal_janeiro, count(Canal_janeiro)-count(Canal_fevereiro) as Saldo, count(Canal_janeiro) as Ativos_jan from canais_janeiro_fevereiro
group by Canal_janeiro);

create table canais_churn_jan_fev
(select Canal_janeiro, Saldo, Ativos_jan, Saldo/Ativos_jan as churn from canais_saldo_jan_fev
order by Canal_janeiro);

create table canais_tempo_de_vida_jan_fev
(select Canal_janeiro, churn as churn_janeiro, 1/churn as tempo_de_vida_jan from canais_churn_jan_fev
group by Canal_janeiro);

# Tabela de churn e tempo de vida médio de todo período
create table canais_tempo_de_vida_out_nov_dez
(select Canal_outubro, churn_outubro, tempo_de_vida_out, churn_novembro, tempo_de_vida_nov from canais_tempo_de_vida_out_nov
join canais_tempo_de_vida_nov_dez
on canais_tempo_de_vida_out_nov.Canal_outubro = canais_tempo_de_vida_nov_dez.Canal_novembro);

create table canais_tempo_de_vida_dez_jan_fev
(select Canal_dezembro, churn_dezembro, tempo_de_vida_dez, churn_janeiro, tempo_de_vida_jan from canais_tempo_de_vida_dez_jan
join canais_tempo_de_vida_jan_fev
on canais_tempo_de_vida_dez_jan.Canal_dezembro = canais_tempo_de_vida_jan_fev.Canal_janeiro);

create table canais_tempo_de_vida_all
(select Canal_outubro as Canal_churn, churn_outubro, tempo_de_vida_out, churn_novembro, tempo_de_vida_nov,
churn_dezembro, tempo_de_vida_dez, churn_janeiro, tempo_de_vida_jan from canais_tempo_de_vida_out_nov_dez
join canais_tempo_de_vida_dez_jan_fev
on canais_tempo_de_vida_out_nov_dez.Canal_outubro = canais_tempo_de_vida_dez_jan_fev.Canal_dezembro);

create table canais_churn_media
(select Canal_churn, (churn_outubro+churn_novembro+churn_dezembro+churn_janeiro)/4 as churn_media 
from canais_tempo_de_vida_all);

# CLUSTERIZAÇÃO CANAIS: CHURN E TEMPO DE VIDA MEDIO

create table canais_churn_e_tempo_de_vida_medio
(select Canal_churn, churn_media, 1/churn_media as tempo_de_vida_medio from canais_churn_media);

# CLUSTERIZAÇÃO CANAIS: TABELA DAS RECEITAS, CUSTOS, LUCRO, CHURN E E TEMPO DE VIDA MÉDIO

create table canais_clusterizacao
(select Canal, total_receitas as receita, total_custos as custos, margem_final as lucro, 
churn_media as churn, tempo_de_vida_medio as tempo_de_vida from canais
join canais_churn_e_tempo_de_vida_medio
on canais.Canal = canais_churn_e_tempo_de_vida_medio.Canal_churn
order by Lucro desc);

# Volume transacionado fevereiro por canal
create table volume_transacionado_fev 
(select distinct Canal, sum(Volume_Transacionado) as Volume_Transacionado from rentabilidadeanalíticab
where Data_Apuração = '2017-02-01'
group by Canal
order by Volume_Transacionado desc);

# Base ativa fevereiro por canal
create table base_ativa_fev
(select distinct Canal_fevereiro, sum(Transacoes_fevereiro) as base_ativa from canais_fevereiro
group by Canal_fevereiro
order by Transacoes_fevereiro desc);

create table volume_e_transacoes_fevereiro
(select Canal, Volume_Transacionado, base_ativa from volume_transacionado_fev
join base_ativa_fev
on volume_transacionado_fev.Canal = base_ativa_fev.Canal_fevereiro);

# Canal <> Canal 3
create table volume_e_transacoes_marco_canal_1_2
select Canal, Volume_Transacionado as FEV_Volume_Transacionado, base_ativa as FEV_base_ativa, 
Volume_Transacionado as MAR_Volume_Transacionado, base_ativa*1.02 as MAR_base_ativa 
from volume_e_transacoes_fevereiro
where Canal <> 'Canal 3';

create table volume_e_transacoes_abril_canal_1_2
select Canal, FEV_Volume_Transacionado, FEV_base_ativa, MAR_Volume_Transacionado, MAR_base_ativa,
MAR_Volume_Transacionado as ABR_Volume_Transacionado, MAR_base_ativa*1.02 as ABR_base_ativa
from volume_e_transacoes_marco_canal_1_2
where Canal <> 'Canal 3';

create table volume_e_transacoes_maio_canal_1_2
select Canal, FEV_Volume_Transacionado, FEV_base_ativa, MAR_Volume_Transacionado, MAR_base_ativa, ABR_Volume_Transacionado, ABR_base_ativa, 
ABR_Volume_Transacionado as MAI_Volume_Transacionado, ABR_base_ativa*1.02 as MAI_base_ativa 
from volume_e_transacoes_abril_canal_1_2
where Canal <> 'Canal 3';

# Canal = Canal 3
create table volume_e_transacoes_marco_canal_3
select Canal as Canal1, Volume_Transacionado as FEV_Volume_Transacionado, base_ativa as FEV_base_ativa, 
Volume_Transacionado*1.03 as MAR_Volume_Transacionado, base_ativa*1.05 as MAR_base_ativa 
from volume_e_transacoes_fevereiro
where Canal = 'Canal 3';

create table volume_e_transacoes_abril_canal_3
select Canal1, FEV_Volume_Transacionado, FEV_base_ativa, MAR_Volume_Transacionado, MAR_base_ativa,
MAR_Volume_Transacionado*1.03 as ABR_Volume_Transacionado, MAR_base_ativa*1.05 as ABR_base_ativa
from volume_e_transacoes_marco_canal_3
where Canal1 = 'Canal 3';

create table volume_e_transacoes_maio_canal_3
select Canal1, FEV_Volume_Transacionado, FEV_base_ativa, MAR_Volume_Transacionado, MAR_base_ativa, ABR_Volume_Transacionado, ABR_base_ativa, 
ABR_Volume_Transacionado*1.03 as MAI_Volume_Transacionado, ABR_base_ativa*1.05 as MAI_base_ativa 
from volume_e_transacoes_abril_canal_3
where Canal1 = 'Canal 3';
