--販売実績
drop table if exists sales_record_summary;
create table sales_record_summary as
select
  hrs.JYGP,
  hrs.GOKI,
  hrs.KIKI,
  hrs.NAIG,
  hrs.SEIH,
  hrs.URKB,
  hrs.KENS,
  hrs.KISC,
  hrs.EKIS,
  hrs.URGP, --売上年月日
  hrs.KNGP,
  hrs.SEEI,
  hrs.URSU,
  hrs.URKN,
  hrs.SRYM,
  hrs.CODEX,
  hrs.KOUM,
  hrs.KOKU,
  hrs.URI_JIGYO_BA,
	hrs.CATGRY_CD,
  hrs.VAL_CHAIN_KBN_NM,
  hrs.AFTER_BUZ_NM,
  hrs.KNGP_GOKI,
  hrs.LEASE_KBN,
  hrs.PROJECT_NO,
  hrs.PROJECT_NM,
  jt.jigyoumei,
  ncm.TORI_SEI_KANJI,
  ccm.chsh,
  fcm.fknm,
  vkt.val_ktd,
  gcm.GYOSHU_SHO_NM,
  gcm.GYOSHU_DAI_NM,
  substring(hrs.GOKI, 1,2) as check_jsp,
  case
    when substr(hrs.GOKI, 1, 2) in ('S3','S5','S7','S9') then '工事あり'
    when substr(hrs.GOKI, 1, 2) in ('S2','S4','S6','S8','ST') then '工事なし'
    else 'その他' end as koji_check,
  case
    when substr(hrs.GOKI, 1, 2) in ('S2') then '1'
    when substr(hrs.GOKI, 1, 2) in ('S4') then '2'
    when substr(hrs.GOKI, 1, 2) in ('S6') then '3'
    when substr(hrs.GOKI, 1, 2) in ('S8') then '4'
    when substr(hrs.GOKI, 1, 2) in ('ST') then '5'
    when substr(hrs.GOKI, 1, 2) in ('S3') then '6'
    when substr(hrs.GOKI, 1, 2) in ('S5') then '7'
    when substr(hrs.GOKI, 1, 2) in ('S7') then '8'
    when substr(hrs.GOKI, 1, 2) in ('S9') then '9'
	else 'その他' end as sp_order,
  case when extract(month from hrs.URGP) < 4 then extract(year from hrs.URGP) -1
    else extract(year from hrs.URGP) end as fy,
  case
    when hrs.URGP >= '2017-04-01' and hrs.URGP < '2017-10-01' then '17上'
    when hrs.URGP >= '2017-10-01' and hrs.URGP < '2018-04-01 'then '17下'
    when hrs.URGP >= '2018-04-01' and hrs.URGP < '2018-10-01' then '18上'
    when hrs.URGP >= '2018-10-01' and hrs.URGP < '2019-04-01' then '18下'
    when hrs.URGP >= '2019-04-01' and hrs.URGP < '2019-10-01' then '19上'
    when hrs.URGP >= '2019-10-01' and hrs.URGP < '2020-04-01' then '19下'
    when hrs.URGP >= '2020-04-01' and hrs.URGP < '2020-10-01' then '20上'
    when hrs.URGP >= '2020-10-01' and hrs.URGP < '2021-04-01' then '20下'
    else '' end as URGP_half_year,
  to_char(hrs.URGP, 'mm') as URGP_mm,
  to_char(hrs.URGP, 'yyyy-mm') as URGP_yyyymm,
  case
    when to_char(hrs.URGP, 'mm') like '04' then 1
    when to_char(hrs.URGP, 'mm') like '05' then 2
    when to_char(hrs.URGP, 'mm') like '06' then 3
    when to_char(hrs.URGP, 'mm') like '07' then 4
    when to_char(hrs.URGP, 'mm') like '08' then 5
    when to_char(hrs.URGP, 'mm') like '09' then 6
    when to_char(hrs.URGP, 'mm') like '10' then 7
    when to_char(hrs.URGP, 'mm') like '11' then 8
    when to_char(hrs.URGP, 'mm') like '12' then 9
    when to_char(hrs.URGP, 'mm') like '01' then 10
    when to_char(hrs.URGP, 'mm') like '02' then 11
    when to_char(hrs.URGP, 'mm') like '03' then 12
    else null end as month_order

from(
select
  cast(case
		when length(replace(JYGP, ' ',''))=0 then null
	  else JYGP end as date),
  GOKI,
  JYCD,
  NOCD,
  KIKI,
  CHIK,
  NAIG,
  SEIH,
  URKB,
  KENS,
  GYOS,
  KISC,
  EKIS,
  cast(case
	  when length(replace(URGP, ' ',''))=0 then null
	  else URGP end as date),
  cast(case
	  when length(replace(KNGP,  ' ',''))=0 then null
	  else KNGP end as date),
  SEEI,
  URSU,
  cast(to_number(URKN, 'FM999,999,999,999.000')as integer)as URKN, --売上金額
  JGCD,
  SRYM,
  CODEX,
  FUKN,
  KOUM,
  KOKU,
  URI_JIGYO_BA,
  VAL_CHAIN_KBN_CD,
	substring(VAL_CHAIN_KBN_CD from 1 for 2) as CATGRY_CD,
  VAL_CHAIN_KBN_NM,
  AFTER_BUZ_NM,
  KNGP_GOKI,
  LEASE_KBN,
  PROJECT_NO,
  PROJECT_NM
	
from
  hnbi_record_daily
)as hrs

left join nounyuusaki_code_master as ncm
on hrs.NOCD = ncm.tori_cd

left join jigyou_kubun_table as jt
on hrs.JGCD = jt.jigyou_kubun

left join chiku_code_master as ccm
on hrs.CHIK = ccm.chik

left join fuken_code_master as fcm
on hrs.FUKN = fcm.FKCD

left join valuechain_kubun_table as vkt
on hrs.VAL_CHAIN_KBN_CD = vkt.VAL_CHAIN_KBN_CD

left join gyoushu_code_master as gcm
on hrs.GYOS =gcm.TORI_GYOSHU_CD_NONYU

where
  NAIG = '1' and
  JGCD <> 'K' and
  CATGRY_CD in ('04', '05', '06', '07') and
	to_char(URGP, 'yyyy-mm') >= to_char(current_timestamp + '-1months', 'yyyy-mm')
;

--月次
with original as(
(select
	val_ktd
  ,sum(case URGP_mm when to_char(current_timestamp+ '-1months', 'mm') then urkn end) as total_premo
	,sum(case URGP_mm when to_char(current_timestamp, 'mm') then urkn end) as total_thismo
from sales_record_summary
where fy = '2020'
group by val_ktd)

union all

(select
	'合計'
  ,sum(case URGP_mm when to_char(current_timestamp+ '-1months', 'mm') then urkn end) as total_premo
	,sum(case URGP_mm when to_char(current_timestamp, 'mm') then urkn end) as total_thismo
from sales_record_summary
where fy = '2020'))
select
	val_ktd
  ,round(cast(total_premo as numeric)/1000000,0) as total_premo
  ,round(cast(total_thismo as numeric)/1000000,0) as total_thismo
from original order by val_ktd;

"val_ktd","total_premo","total_thismo"
"1.修理","35","44"
"2.パーツ","277","294"
"3.OH","71","49"
"4.改造","280","20"
"合計","664","408"
