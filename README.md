
/* 
引合情報に関係するテーブルを作成するsql文
以下、作成するテーブル
1.引合情報サマリー
2.年度別・月次累積
3.活動系別・半期
4.活動系別・本年度・月次
5.活動系別・前年度・月次
5.引合ランク別・本年度・月次
6.引合ランク別・前年度・月次
*/

/* 
メモ
サマリーテーブル作成条件
カテゴリー区分：04, 05, 06, 07
内外区分：1
製品種別名称：研削盤・切削機・ﾏｼﾆﾝｸﾞ 

*/

--1.引合情報サマリー
drop table if exists inquiry_record_summary;
create table inquiry_record_summary as
(
select 
  ush.hikiai_no, --引合no
  ush.hikiai_date, --引合日
  ush.shain_nm, --担当者名
  ush.chiku_nm, --地区名称
  ush.sehin_shubetsu_nm, --製品種別名称
  ush.tori_nm_nonyu_sei, --取引先名称(納入先)正式
  ush.todofkn_nm_jp, --都道府県、国
  ush.kibo_nm, --規模名称
  ush.kokunaigai_kbn_nm, --国内外区分名称
  ush.tori_gyoshu_cd_nonyu, --業種コード（納入先）
  ush.gyoshu_sho_nm, --業種詳細名称
  ush.naihan_gaihan_kbn_cd, --内販外販区分名称
  ush.kishu_cd, --機種コード
  ush.sehin_nm_eigyo, --製品名称（営業）
  ush.hikiai_rank_cd, --引合ランクコード
  ush.juchu_yotei_kin, --受注予定金額
  ush.kachimake_kbn_cd, --勝負区分コード
  ush.val_chain_kbn_cd, --バリューチェーン区分コード
  ush.val_chain_kbn_nm, --バリューチェーン区分名称
  ush.after_buz_nm, --アフタービジネス名称
	ush.val_category,
	vkt.val_ktd, --活動系名称
	concat(ush.hikiai_no, '-', vkt.val_ktd) as hikiai_no_val_ktd,
  case when extract(month from ush.hikiai_date) < 4 then extract(year from ush.hikiai_date) -1
    else extract(year from ush.hikiai_date) end as fy,
  case
    when ush.hikiai_date >= '2017-04-01' and ush.hikiai_date < '2017-10-01' then '17上'
    when ush.hikiai_date >= '2017-10-01' and ush.hikiai_date < '2018-04-01 'then '17下'
    when ush.hikiai_date >= '2018-04-01' and ush.hikiai_date < '2018-10-01' then '18上'
    when ush.hikiai_date >= '2018-10-01' and ush.hikiai_date < '2019-04-01' then '18下'
    when ush.hikiai_date >= '2019-04-01' and ush.hikiai_date < '2019-10-01' then '19上'
    when ush.hikiai_date >= '2019-10-01' and ush.hikiai_date < '2020-04-01' then '19下'
    when ush.hikiai_date >= '2020-04-01' and ush.hikiai_date < '2020-10-01' then '20上'
    when ush.hikiai_date >= '2020-10-01' and ush.hikiai_date < '2021-04-01' then '20下'
    else '' end as hikiai_date_half_year,
  to_char(ush.hikiai_date, 'mm') as hikiai_date_mm,
  to_char(ush.hikiai_date, 'yyyy-mm') as hikiai_date_yyyymm,
  case
    when to_char(ush.hikiai_date, 'mm') like '04' then 1
    when to_char(ush.hikiai_date, 'mm') like '05' then 2
    when to_char(ush.hikiai_date, 'mm') like '06' then 3
    when to_char(ush.hikiai_date, 'mm') like '07' then 4
    when to_char(ush.hikiai_date, 'mm') like '08' then 5
    when to_char(ush.hikiai_date, 'mm') like '09' then 6
    when to_char(ush.hikiai_date, 'mm') like '10' then 7
    when to_char(ush.hikiai_date, 'mm') like '11' then 8
    when to_char(ush.hikiai_date, 'mm') like '12' then 9
    when to_char(ush.hikiai_date, 'mm') like '01' then 10
    when to_char(ush.hikiai_date, 'mm') like '02' then 11
    when to_char(ush.hikiai_date, 'mm') like '03' then 12
    else null end as month_order,
  case
    when ush.HIKIAI_RANK_CD in ('AS') then 1
    when ush.HIKIAI_RANK_CD in ('AA') then 2
    when ush.HIKIAI_RANK_CD in ('A') then 3
    when ush.HIKIAI_RANK_CD in ('BA') then 4
    when ush.HIKIAI_RANK_CD in ('BB') then 5
    when ush.HIKIAI_RANK_CD in ('B') then 6
    when ush.HIKIAI_RANK_CD in ('C') then 7
    when ush.HIKIAI_RANK_CD in ('D') then 8
    when ush.HIKIAI_RANK_CD in ('ST') then 9
    when ush.HIKIAI_RANK_CD in ('Z') then 10
    else 11 end as hikiai_rank_order
from (
	select
	  hikiai_no, --引合no
    cast(case
	    when length(replace(hikiai_date, ' ',''))=0 then null
	    else hikiai_date end as date), --引合日
    shain_nm, --担当者名
    chiku_nm, --地区名称
    trim(sehin_shubetsu_nm) as sehin_shubetsu_nm, --製品種別名称
    tori_nm_nonyu_sei, --取引先名称(納入先)正式
    todofkn_nm_jp, --都道府県、国
    kibo_nm, --規模名称
    kokunaigai_kbn_nm, --国内外区分名称
    tori_gyoshu_cd_nonyu, --業種コード（納入先）
    gyoshu_sho_nm, --業種詳細名称
    naihan_gaihan_kbn_cd, --内販外販区分
    kishu_cd, --機種コード
    sehin_nm_eigyo, --製品名称（営業）
    trim(hikiai_rank_cd) as hikiai_rank_cd, --引合ランクコード
    cast(to_number(juchu_yotei_kin, 'fm999,999,999,999.000')as integer)as juchu_yotei_kin, --受注予定金額  
    trim(kachimake_kbn_cd) as kachimake_kbn_cd, --勝負区分コード
    trim(val_chain_kbn_cd) as val_chain_kbn_cd, --バリューチェーン区分コード
    val_chain_kbn_nm, --バリューチェーン区分名称
    after_buz_nm, --アフタービジネス名称
	  substr(val_chain_kbn_cd, 1,2) as val_category
	from
		shinki_hikiai_17km_19km
	where	
		trim(sehin_shubetsu_nm)  in  ('マシニングセンタ', '研削盤', '切削機') and
		naihan_gaihan_kbn_cd like '1' and
		substr(trim(val_chain_kbn_cd), 1,2) in ('04', '05', '06', '07')

	union all
		
	select
	hikiai_no, --引合no
    cast(case
		 when length(replace(hikiai_date, ' ',''))=0 then null
	else hikiai_date
	end
	as date), --引合日
    shain_nm, --担当者名
    chiku_nm, --地区名称
    trim(sehin_shubetsu_nm) as sehin_shubetsu_nm , --製品種別名称
    tori_nm_nonyu_sei, --取引先名称(納入先)正式
    todofkn_nm_jp, --都道府県、国
    kibo_nm, --規模名称
    kokunaigai_kbn_nm, --国内外区分名称
    tori_gyoshu_cd_nonyu, --業種コード（納入先）
    gyoshu_sho_nm, --業種詳細名称
    naihan_gaihan_kbn_cd, --内販外販区分
    kishu_cd, --機種コード
    sehin_nm_eigyo, --製品名称（営業）
    trim(hikiai_rank_cd) as hikiai_rank_cd, --引合ランクコード
    cast(to_number(juchu_yotei_kin, 'fm999,999,999,999.000')as integer)as juchu_yotei_kin, --受注予定金額  
    trim(kachimake_kbn_cd) as kachimake_kbn_cd, --勝負区分コード
    trim(val_chain_kbn_cd) as val_chain_kbn_cd, --バリューチェーン区分コード
    val_chain_kbn_nm, --バリューチェーン区分名称
    after_buz_nm, --アフタービジネス名称
	substr(val_chain_kbn_cd, 1,2) as val_category
	from
		shinki_hikiai_19sm
	where
		trim(sehin_shubetsu_nm)  in  ('マシニングセンタ', '研削盤', '切削機') and
		naihan_gaihan_kbn_cd like '1' and
		substr(trim(val_chain_kbn_cd), 1,2) in ('04', '05', '06', '07')
	)as ush
	
left join valuechain_kubun_table as vkt
on ush.val_chain_kbn_cd = vkt.val_chain_kbn_cd
	
);

--2.年度別・月次累積

select
	hikiai_date_mm
	,fy
	,case when fy = 2017 then sum(count(hikiai_no_val_ktd)) over(partition by fy order by month_order rows unbounded preceding) end as agg_17
	,case when fy = 2018 then sum(count(hikiai_no_val_ktd)) over(partition by fy order by month_order rows unbounded preceding) end as agg_18
	,case when fy = 2019 then sum(count(hikiai_no_val_ktd)) over(partition by fy order by month_order rows unbounded preceding) end as agg_19
	,case when fy = 2020 then sum(count(hikiai_no_val_ktd)) over(partition by fy order by month_order rows unbounded preceding) end as agg_20
from inquiry_record_summary
where fy in ('2017','2018', '2019', '2020') and hikiai_rank_cd in ('AS', 'AA', 'A', 'BA')
group by hikiai_date_mm, fy, month_order
order by fy, month_order

"hikiai_date_mm","fy","agg_17","agg_18","agg_19","agg_20"
"04","2017","200",NULL,NULL,NULL
"05","2017","472",NULL,NULL,NULL
"06","2017","699",NULL,NULL,NULL
"07","2017","901",NULL,NULL,NULL
"08","2017","1250",NULL,NULL,NULL
"09","2017","1513",NULL,NULL,NULL
"10","2017","1717",NULL,NULL,NULL
"11","2017","1958",NULL,NULL,NULL
"12","2017","2402",NULL,NULL,NULL
"01","2017","2577",NULL,NULL,NULL
"02","2017","2773",NULL,NULL,NULL
"03","2017","2963",NULL,NULL,NULL
"04","2018",NULL,"196",NULL,NULL
"05","2018",NULL,"446",NULL,NULL
"06","2018",NULL,"639",NULL,NULL
"07","2018",NULL,"922",NULL,NULL
"08","2018",NULL,"1143",NULL,NULL
"09","2018",NULL,"1509",NULL,NULL
"10","2018",NULL,"1763",NULL,NULL
"11","2018",NULL,"2021",NULL,NULL
"12","2018",NULL,"2207",NULL,NULL
"01","2018",NULL,"2355",NULL,NULL
"02","2018",NULL,"2563",NULL,NULL
"03","2018",NULL,"2797",NULL,NULL
"04","2019",NULL,NULL,"181",NULL
"05","2019",NULL,NULL,"344",NULL
"06","2019",NULL,NULL,"497",NULL
"07","2019",NULL,NULL,"670",NULL
"08","2019",NULL,NULL,"956",NULL
"09","2019",NULL,NULL,"1202",NULL
"10","2019",NULL,NULL,"1341",NULL
"11","2019",NULL,NULL,"1556",NULL
"12","2019",NULL,NULL,"1830",NULL
"01","2019",NULL,NULL,"2002",NULL
"02","2019",NULL,NULL,"2173",NULL
"03","2019",NULL,NULL,"2639",NULL
"04","2020",NULL,NULL,NULL,"163"
"05","2020",NULL,NULL,NULL,"286"
"06","2020",NULL,NULL,NULL,"430"
"07","2020",NULL,NULL,NULL,"651"
"08","2020",NULL,NULL,NULL,"790"
"09","2020",NULL,NULL,NULL,"893"

--3.活動系別・半期
(select
	val_ktd
	,count(case hikiai_date_half_year when '17上' then hikiai_rank_cd end) as total_17kami
	,count(case hikiai_date_half_year when '17下' then hikiai_rank_cd end) as total_17shimo
	,count(case hikiai_date_half_year when '18上' then hikiai_rank_cd end) as total_18kami
	,count(case hikiai_date_half_year when '18下' then hikiai_rank_cd end) as total_18shimo
	,count(case hikiai_date_half_year when '19上' then hikiai_rank_cd end) as total_19kami
	,count(case hikiai_date_half_year when '19下' then hikiai_rank_cd end) as total_19shimo
	,count(case hikiai_date_half_year when '20上' then hikiai_rank_cd end) as total_20kami
	,count(case hikiai_date_half_year when '20下' then hikiai_rank_cd end) as total_20shimo
from inquiry_record_summary
where hikiai_rank_cd in ('AS', 'AA', 'A', 'BA')
group by val_ktd
order by val_ktd)

union all

(select
	'合計'
	,count(case hikiai_date_half_year when '17上' then hikiai_rank_cd end) as total_17kami
	,count(case hikiai_date_half_year when '17下' then hikiai_rank_cd end) as total_17shimo
	,count(case hikiai_date_half_year when '18上' then hikiai_rank_cd end) as total_18kami
	,count(case hikiai_date_half_year when '18下' then hikiai_rank_cd end) as total_18shimo
	,count(case hikiai_date_half_year when '19上' then hikiai_rank_cd end) as total_19kami
	,count(case hikiai_date_half_year when '19下' then hikiai_rank_cd end) as total_19shimo
	,count(case hikiai_date_half_year when '20上' then hikiai_rank_cd end) as total_20kami
	,count(case hikiai_date_half_year when '20下' then hikiai_rank_cd end) as total_20shimo
from inquiry_record_summary
where hikiai_rank_cd in ('AS', 'AA', 'A', 'BA'))

"val_ktd","total_17kami","total_17shimo","total_18kami","total_18shimo","total_19kami","total_19shimo","total_20kami","total_20shimo"
"1.修理","3","2","0","2","0","1","1","0"
"2.パーツ","431","416","513","491","405","418","231","0"
"3.OH","237","202","235","184","191","184","314","0"
"4.改造","842","830","761","611","606","834","347","0"
"合計","1513","1450","1509","1288","1202","1437","893","0"

--4.活動系別・本年度・月次

(select
	val_ktd
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2020' and hikiai_rank_cd in ('AS', 'AA', 'A', 'BA')
group by val_ktd
order by val_ktd)

union all

(select
	'合計'
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2020' and hikiai_rank_cd in ('AS', 'AA', 'A', 'BA'))

"val_ktd","total_04","total_05","total_06","total_07","total_08","total_09","total_10","total_11","total_12","total_01","total_02","total_03"
"1.修理","1","0","0","0","0","0","0","0","0","0","0","0"
"2.パーツ","48","31","34","46","44","28","0","0","0","0","0","0"
"3.OH","60","15","48","116","46","29","0","0","0","0","0","0"
"4.改造","54","77","62","59","49","46","0","0","0","0","0","0"
"合計","163","123","144","221","139","103","0","0","0","0","0","0"

--5.活動系別・前年度・月次

(select
	val_ktd
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2019' and hikiai_rank_cd in ('AS', 'AA', 'A', 'BA')
group by val_ktd
order by val_ktd)

union all

(select
	'合計'
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2019' and hikiai_rank_cd in ('AS', 'AA', 'A', 'BA'))

"val_ktd","total_04","total_05","total_06","total_07","total_08","total_09","total_10","total_11","total_12","total_01","total_02","total_03"
"1.修理","0","0","0","0","0","0","0","1","0","0","0","0"
"2.パーツ","75","57","63","61","81","68","48","62","65","53","50","140"
"3.OH","50","29","25","27","36","24","8","32","27","43","28","46"
"4.改造","56","77","65","85","169","154","83","120","182","76","93","280"
"合計","181","163","153","173","286","246","139","215","274","172","171","466"

--6.引合ランク別・本年度・月次

with original as(
(select
	hikiai_rank_cd
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2020'
group by hikiai_rank_cd)

union all

(select
	'合計'
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2020'))
select
    *,
    case
    when HIKIAI_RANK_CD in ('AS') then 1
    when HIKIAI_RANK_CD in ('AA') then 2
    when HIKIAI_RANK_CD in ('A') then 3
    when HIKIAI_RANK_CD in ('BA') then 4
    when HIKIAI_RANK_CD in ('BB') then 5
    when HIKIAI_RANK_CD in ('B') then 6
    when HIKIAI_RANK_CD in ('C') then 7
    when HIKIAI_RANK_CD in ('D') then 8
    when HIKIAI_RANK_CD in ('ST') then 9
    when HIKIAI_RANK_CD in ('Z') then 10
    else 11 end as hikiai_rank_order
from original
order by hikiai_rank_order

"hikiai_rank_cd","total_04","total_05","total_06","total_07","total_08","total_09","total_10","total_11","total_12","total_01","total_02","total_03","hikiai_rank_order"
"AS","0","0","1","0","1","0","0","0","0","0","0","0",1
"AA","72","27","56","48","47","31","0","0","0","0","0","0",2
"A","25","32","26","37","46","30","0","0","0","0","0","0",3
"BA","66","64","61","136","45","42","0","0","0","0","0","0",4
"BB","1","9","50","23","51","35","0","0","0","0","0","0",5
"B","95","50","114","96","72","99","0","0","0","0","0","0",6
"C","29","31","8","17","9","5","0","0","0","0","0","0",7
"D","6","1","2","2","0","0","0","0","0","0","0","0",8
"Z","0","0","3","1","12","0","0","0","0","0","0","0",10
"合計","294","214","321","360","283","242","0","0","0","0","0","0",11

--7.引合ランク別・前年度・月次

with original as(
(select
	hikiai_rank_cd
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2019'
group by hikiai_rank_cd)

union all

(select
	'合計'
	,count(case hikiai_date_mm when '04' then hikiai_rank_cd end) as total_04
	,count(case hikiai_date_mm when '05' then hikiai_rank_cd end) as total_05
	,count(case hikiai_date_mm when '06' then hikiai_rank_cd end) as total_06
	,count(case hikiai_date_mm when '07' then hikiai_rank_cd end) as total_07
	,count(case hikiai_date_mm when '08' then hikiai_rank_cd end) as total_08
	,count(case hikiai_date_mm when '09' then hikiai_rank_cd end) as total_09
	,count(case hikiai_date_mm when '10' then hikiai_rank_cd end) as total_10
	,count(case hikiai_date_mm when '11' then hikiai_rank_cd end) as total_11
	,count(case hikiai_date_mm when '12' then hikiai_rank_cd end) as total_12
	,count(case hikiai_date_mm when '01' then hikiai_rank_cd end) as total_01
	,count(case hikiai_date_mm when '02' then hikiai_rank_cd end) as total_02
	,count(case hikiai_date_mm when '03' then hikiai_rank_cd end) as total_03
from inquiry_record_summary
where fy = '2019'))
select
    *,
    case
    when HIKIAI_RANK_CD in ('AS') then 1
    when HIKIAI_RANK_CD in ('AA') then 2
    when HIKIAI_RANK_CD in ('A') then 3
    when HIKIAI_RANK_CD in ('BA') then 4
    when HIKIAI_RANK_CD in ('BB') then 5
    when HIKIAI_RANK_CD in ('B') then 6
    when HIKIAI_RANK_CD in ('C') then 7
    when HIKIAI_RANK_CD in ('D') then 8
    when HIKIAI_RANK_CD in ('ST') then 9
    when HIKIAI_RANK_CD in ('Z') then 10
    else 11 end as hikiai_rank_order
from original
order by hikiai_rank_order

"hikiai_rank_cd","total_04","total_05","total_06","total_07","total_08","total_09","total_10","total_11","total_12","total_01","total_02","total_03","hikiai_rank_order"
"AS","0","2","1","0","3","1","1","0","0","0","0","0",1
"AA","97","76","70","67","161","96","68","81","141","88","54","68",2
"A","52","53","43","71","76","60","40","96","77","32","50","69",3
"BA","32","32","39","35","46","89","30","38","56","52","67","329",4
"BB","0","0","0","0","0","0","0","0","2","5","5","23",5
"B","83","99","111","90","116","96","82","88","90","113","85","216",6
"C","47","37","51","38","20","15","39","23","44","65","38","56",7
"D","7","8","15","6","11","2","3","10","3","5","9","5",8
"ST","1","0","0","0","0","0","0","0","0","0","0","0",9
"Z","1","1","3","6","1","1","1","5","1","1","2","0",10
"合計","320","308","333","313","434","360","264","341","414","361","310","766",11

--
order
--
--サマリーテーブル
drop table order_record_summary;
create table order_record_summary as
select
    uor.GOKI,
    uor.JYUR,
    uor.KIKI,
    uor.GYOS,
    uor.KISC,
    uor.EKIS,
    uor.SEIH,
    uor.NAIG,
    uor.KETE,
    uor.NOYM,
    uor.URKB,
    uor.KENSHU_YOTEI_DATE,
    uor.JYKN, 
    uor.MITSU_SEIZO_GENKA, 
    uor.KOKUNAIGAI_KBN,
    uor.VAL_CHAIN_KBN_NM,
    jt.jigyoumei,
    ncm.TORI_SEI_KANJI,
    ccm.chsh,
    fcm.fknm,
    vkt.val_ktd,
    vkt.val_category,
    gcm.GYOSHU_SHO_NM,
    gcm.GYOSHU_DAI_NM,
    case when extract(month from uor.jyur) < 4 then extract(year from uor.jyur) -1
    else extract(year from uor.jyur) end as fy,
    substr(uor.goki, 1, 2) as check_sp,
    case
        when uor.KOKUNAIGAI_KBN = '1:KOKUNAI ' then '国内'
        when uor.KOKUNAIGAI_KBN = '2:KAIGAI  ' then '海外'
        else 'その他' end  as kokunai_kaigai_flag,
    case
        when substr(uor.goki, 1, 2) in ('S3','S5','S7','S9') then '工事あり'
        when substr(uor.goki, 1, 2) in ('S2','S4','S6','S8','ST') then '工事なし'
        else 'その他' end as koji_check,
    case
        when substr(uor.goki, 1, 2) in ('S2') then '1'
        when substr(uor.goki, 1, 2) in ('S4') then '2'
        when substr(uor.goki, 1, 2) in ('S6') then '3'
        when substr(uor.goki, 1, 2) in ('S8') then '4'
        when substr(uor.goki, 1, 2) in ('ST') then '5'
        when substr(uor.goki, 1, 2) in ('S3') then '6'
        when substr(uor.goki, 1, 2) in ('S5') then '7'
        when substr(uor.goki, 1, 2) in ('S7') then '8'
        when substr(uor.goki, 1, 2) in ('S9') then '9'
        else 'その他' end as sp_order,
      case
    when uor.jyur >= '2017-04-01' and uor.jyur < '2017-10-01' then '17上'
    when uor.jyur >= '2017-10-01' and uor.jyur < '2018-04-01 'then '17下'
    when uor.jyur >= '2018-04-01' and uor.jyur < '2018-10-01' then '18上'
    when uor.jyur >= '2018-10-01' and uor.jyur < '2019-04-01' then '18下'
    when uor.jyur >= '2019-04-01' and uor.jyur < '2019-10-01' then '19上'
    when uor.jyur >= '2019-10-01' and uor.jyur < '2020-04-01' then '19下'
    when uor.jyur >= '2020-04-01' and uor.jyur < '2020-10-01' then '20上'
    when uor.jyur >= '2020-10-01' and uor.jyur < '2021-04-01' then '20下'
    else '' end as jyur_half_year,
  to_char(uor.jyur, 'mm') as jyur_mm,
  to_char(uor.jyur, 'yyyy-mm') as jyur_yyyymm,
  case
    when to_char(uor.jyur, 'mm') like '04' then 1
    when to_char(uor.jyur, 'mm') like '05' then 2
    when to_char(uor.jyur, 'mm') like '06' then 3
    when to_char(uor.jyur, 'mm') like '07' then 4
    when to_char(uor.jyur, 'mm') like '08' then 5
    when to_char(uor.jyur, 'mm') like '09' then 6
    when to_char(uor.jyur, 'mm') like '10' then 7
    when to_char(uor.jyur, 'mm') like '11' then 8
    when to_char(uor.jyur, 'mm') like '12' then 9
    when to_char(uor.jyur, 'mm') like '01' then 10
    when to_char(uor.jyur, 'mm') like '02' then 11
    when to_char(uor.jyur, 'mm') like '03' then 12
    else null end as month_order
    
from(
select
    GOKI,
    cast(case
		 when length(JYUR)=0 then null
		 else JYUR
		 end
         as date),
    NOCD,
    JIKB,
    CHIK,
    KIKI,
    GYOS,
    KISC,
    EKIS,
    SEIH,
    NAIG,
    KETE,
    NOYM,
    URKB,
	cast(case
	when length(replace(KENSHU_YOTEI_DATE, ' ',''))=0 then null
	else KENSHU_YOTEI_DATE
	end
	as date),
    cast(to_number(JYKN, 'FM999,999,999,999.000')as integer)as JYKN,
    cast(to_number(MITSU_SEIZO_GENKA, 'FM999,999,999,999.000')as integer)as MITSU_SEIZO_GENKA,
    TODOFKN_CD,
    KOKUNAIGAI_KBN,
    VAL_CHAIN_KBN_CD,
    VAL_CHAIN_KBN_NM
from order_record_17kami_19shimo

union all

select
    GOKI,
    cast(case
		 when length(JYUR)=0 then null
		 else JYUR
		 end
         as date),
    NOCD,
    JIKB,
    CHIK,
    KIKI,
    GYOS,
    KISC,
    EKIS,
    SEIH,
    NAIG,
    KETE,
    NOYM,
    URKB,
	cast(case
	when length(replace(KENSHU_YOTEI_DATE, ' ',''))=0 then null
	else KENSHU_YOTEI_DATE
	end
	as date),
    cast(to_number(JYKN, 'FM999,999,999,999.000')as integer)as JYKN,
    cast(to_number(MITSU_SEIZO_GENKA, 'FM999,999,999,999.000')as integer)as MITSU_SEIZO_GENKA,
    TODOFKN_CD,
    KOKUNAIGAI_KBN,
    VAL_CHAIN_KBN_CD,
    VAL_CHAIN_KBN_NM
from order_record_20kami_lamo
--where substr(jyur, 1,6) <> to_char(current_timestamp + '-1months', 'yyyymm')
where substr(jyur, 1,6) <> to_char(current_timestamp + '-1months', 'yyyymm')

union all

select
    GOKI,
    cast(case
		 when length(JYUR)=0 then null
		 else JYUR
		 end
         as date),
    NOCD,
    JIKB,
    CHIK,
    KIKI,
    GYOS,
    KISC,
    EKIS,
    SEIH,
    NAIG,
    KETE,
    NOYM,
    URKB,
	cast(case
	when length(replace(KENSHU_YOTEI_DATE, ' ',''))=0 then null
	else KENSHU_YOTEI_DATE
	end
	as date),
    cast(to_number(JYKN, 'FM999,999,999,999.000')as integer)as JYKN,
    cast(to_number(MITSU_SEIZO_GENKA, 'FM999,999,999,999.000')as integer)as MITSU_SEIZO_GENKA,
    TODOFKN_CD,
    KOKUNAIGAI_KBN,
    VAL_CHAIN_KBN_CD,
    VAL_CHAIN_KBN_NM
from order_record_lamo_thmo
--where substr(jyur, 1,6) = to_char(now(), 'yyyymm') 

)as uor

left join nounyuusaki_code_master as ncm
on uor.nocd = ncm.tori_cd

left join jigyou_kubun_table as jt
on uor.jikb = jt.jigyou_kubun

left join chiku_code_master as ccm
on uor.chik = ccm.chik

left join fuken_code_master as fcm
on uor.TODOFKN_CD = fcm.FKCD

left join valuechain_kubun_table as vkt
on uor.VAL_CHAIN_KBN_CD = vkt.VAL_CHAIN_KBN_CD

left join gyoushu_code_master as gcm
on uor.gyos =gcm.TORI_GYOSHU_CD_NONYU

where 
    uor.JIKB <> 'K' and
    uor.naig = '1' and
    vkt.val_category in ('04', '05', '06', '07')
;

--上期・累積

with original as(
select
	jyur_mm
	,jyur_half_year
	,case when jyur_half_year like '17上' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_17kami
	,case when jyur_half_year like '18上' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_18kami
	,case when jyur_half_year like '19上' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_19kami
	,case when jyur_half_year like '20上' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_20kami
from order_record_summary
where jyur_half_year in ('17上','18上', '19上', '20上')
group by jyur_mm, jyur_half_year, month_order
order by jyur_half_year, month_order)
select
	jyur_mm
	,jyur_half_year
	,round(agg_17kami/1000000,1) as agg_17kami
	,round(agg_18kami/1000000,1) as agg_18kami
	,round(agg_19kami/1000000,1) as agg_19kami
	,round(agg_20kami/1000000,1) as agg_20kami
from original


--下期・累積
with original as(
select
	jyur_mm
	,jyur_half_year
	,case when jyur_half_year like '17下' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_17shimo
	,case when jyur_half_year like '18下' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_18shimo
	,case when jyur_half_year like '19下' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_19shimo
	,case when jyur_half_year like '20下' then sum(sum(jykn)) over(partition by jyur_half_year order by month_order rows unbounded preceding) end as agg_20shimo
from order_record_summary
where jyur_half_year in ('17下','18下', '19下', '20下')
group by jyur_mm, jyur_half_year, month_order
order by jyur_half_year, month_order)
select
	jyur_mm
	,jyur_half_year
	,round(agg_17shimo/1000000,1) as agg_17shimo
	,round(agg_18shimo/1000000,1) as agg_18shimo
	,round(agg_19shimo/1000000,1) as agg_19shimo
	,round(agg_20shimo/1000000,1) as agg_20shimo
from original

--半期・推移

with original as(
(select
	val_ktd
	,sum(case jyur_half_year when '17上' then jykn end) as total_17kami
	,sum(case jyur_half_year when '17下' then jykn end) as total_17shimo
	,sum(case jyur_half_year when '18上' then jykn end) as total_18kami
	,sum(case jyur_half_year when '18下' then jykn end) as total_18shimo
	,sum(case jyur_half_year when '19上' then jykn end) as total_19kami
	,sum(case jyur_half_year when '19下' then jykn end) as total_19shimo
	,sum(case jyur_half_year when '20上' then jykn end) as total_20kami
	,sum(case jyur_half_year when '20下' then jykn end) as total_20shimo
from order_record_summary
group by val_ktd)

union all

(select
	'合計'
	,sum(case jyur_half_year when '17上' then jykn end) as total_17kami
	,sum(case jyur_half_year when '17下' then jykn end) as total_17shimo
	,sum(case jyur_half_year when '18上' then jykn end) as total_18kami
	,sum(case jyur_half_year when '18下' then jykn end) as total_18shimo
	,sum(case jyur_half_year when '19上' then jykn end) as total_19kami
	,sum(case jyur_half_year when '19下' then jykn end) as total_19shimo
	,sum(case jyur_half_year when '20上' then jykn end) as total_20kami
	,sum(case jyur_half_year when '20下' then jykn end) as total_20shimo
from order_record_summary))
select
	val_ktd
    ,round(cast(total_17kami as numeric)/1000000,0) as total_17kami
    ,round(cast(total_17shimo as numeric)/1000000,0) as total_17shimo
    ,round(cast(total_18kami as numeric)/1000000,0) as total_18kami
    ,round(cast(total_18shimo as numeric)/1000000,0) as total_18shimo
    ,round(cast(total_19kami as numeric)/1000000,0) as total_19kami
    ,round(cast(total_19shimo as numeric)/1000000,0) as total_19shimo
	,round(cast(total_20kami as numeric)/1000000,0) as total_20kami
    ,round(cast(total_20shimo as numeric)/1000000,0) as total_20shimo
from original

--20年度・月次・推移
with original as (
(select
	val_ktd
	,sum(case jyur_mm when '04' then jykn end) as total_04
	,sum(case jyur_mm when '05' then jykn end) as total_05
	,sum(case jyur_mm when '06' then jykn end) as total_06
	,sum(case jyur_mm when '07' then jykn end) as total_07
	,sum(case jyur_mm when '08' then jykn end) as total_08
	,sum(case jyur_mm when '09' then jykn end) as total_09
	,sum(case jyur_mm when '10' then jykn end) as total_10
	,sum(case jyur_mm when '11' then jykn end) as total_11
	,sum(case jyur_mm when '12' then jykn end) as total_12
	,sum(case jyur_mm when '01' then jykn end) as total_01
	,sum(case jyur_mm when '02' then jykn end) as total_02
	,sum(case jyur_mm when '03' then jykn end) as total_03
from order_record_summary
where fy = '2020'
group by val_ktd)

union all

(select
	'合計'
	,sum(case jyur_mm when '04' then jykn end) as total_04
	,sum(case jyur_mm when '05' then jykn end) as total_05
	,sum(case jyur_mm when '06' then jykn end) as total_06
	,sum(case jyur_mm when '07' then jykn end) as total_07
	,sum(case jyur_mm when '08' then jykn end) as total_08
	,sum(case jyur_mm when '09' then jykn end) as total_09
	,sum(case jyur_mm when '10' then jykn end) as total_10
	,sum(case jyur_mm when '11' then jykn end) as total_11
	,sum(case jyur_mm when '12' then jykn end) as total_12
	,sum(case jyur_mm when '01' then jykn end) as total_01
	,sum(case jyur_mm when '02' then jykn end) as total_02
	,sum(case jyur_mm when '03' then jykn end) as total_03
from order_record_summary
where fy = '2020'))
select
	val_ktd
    ,round(cast(total_04 as numeric)/1000000,1) as total_04
    ,round(cast(total_05 as numeric)/1000000,1) as total_05
    ,round(cast(total_06 as numeric)/1000000,1) as total_06
    ,round(cast(total_07 as numeric)/1000000,1) as total_07
    ,round(cast(total_08 as numeric)/1000000,1) as total_08
    ,round(cast(total_09 as numeric)/1000000,1) as total_09
    ,round(cast(total_10 as numeric)/1000000,1) as total_10
    ,round(cast(total_11 as numeric)/1000000,1) as total_11
    ,round(cast(total_12 as numeric)/1000000,1) as total_12
    ,round(cast(total_01 as numeric)/1000000,1) as total_01
    ,round(cast(total_02 as numeric)/1000000,1) as total_02
    ,round(cast(total_03 as numeric)/1000000,1) as total_03
from original

--19年度・月次・推移

with original as (
(select
	val_ktd
	,sum(case jyur_mm when '04' then jykn end) as total_04
	,sum(case jyur_mm when '05' then jykn end) as total_05
	,sum(case jyur_mm when '06' then jykn end) as total_06
	,sum(case jyur_mm when '07' then jykn end) as total_07
	,sum(case jyur_mm when '08' then jykn end) as total_08
	,sum(case jyur_mm when '09' then jykn end) as total_09
	,sum(case jyur_mm when '10' then jykn end) as total_10
	,sum(case jyur_mm when '11' then jykn end) as total_11
	,sum(case jyur_mm when '12' then jykn end) as total_12
	,sum(case jyur_mm when '01' then jykn end) as total_01
	,sum(case jyur_mm when '02' then jykn end) as total_02
	,sum(case jyur_mm when '03' then jykn end) as total_03
from order_record_summary
where fy = '2019'
group by val_ktd)

union all

(select
	'合計'
	,sum(case jyur_mm when '04' then jykn end) as total_04
	,sum(case jyur_mm when '05' then jykn end) as total_05
	,sum(case jyur_mm when '06' then jykn end) as total_06
	,sum(case jyur_mm when '07' then jykn end) as total_07
	,sum(case jyur_mm when '08' then jykn end) as total_08
	,sum(case jyur_mm when '09' then jykn end) as total_09
	,sum(case jyur_mm when '10' then jykn end) as total_10
	,sum(case jyur_mm when '11' then jykn end) as total_11
	,sum(case jyur_mm when '12' then jykn end) as total_12
	,sum(case jyur_mm when '01' then jykn end) as total_01
	,sum(case jyur_mm when '02' then jykn end) as total_02
	,sum(case jyur_mm when '03' then jykn end) as total_03
from order_record_summary
where fy = '2019'))
select
	val_ktd
    ,round(cast(total_04 as numeric)/1000000,1) as total_04
    ,round(cast(total_05 as numeric)/1000000,1) as total_05
    ,round(cast(total_06 as numeric)/1000000,1) as total_06
    ,round(cast(total_07 as numeric)/1000000,1) as total_07
    ,round(cast(total_08 as numeric)/1000000,1) as total_08
    ,round(cast(total_09 as numeric)/1000000,1) as total_09
    ,round(cast(total_10 as numeric)/1000000,1) as total_10
    ,round(cast(total_11 as numeric)/1000000,1) as total_11
    ,round(cast(total_12 as numeric)/1000000,1) as total_12
    ,round(cast(total_01 as numeric)/1000000,1) as total_01
    ,round(cast(total_02 as numeric)/1000000,1) as total_02
    ,round(cast(total_03 as numeric)/1000000,1) as total_03
from original
