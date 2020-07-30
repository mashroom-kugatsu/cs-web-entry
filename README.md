
登録場所
C:\Users\00183338\Box\DEPT_カスタマーサポート部\150_満足度調査 お客様相談センター管理\web返信
-----
受注残SQL
--受注残--
--\encoding UTF8;

drop table order_backlog
;
create table order_backlog(
    JUCHU_DATE  text,
    GOKI text,
    J_TORI_RYA_KANJI text,
    JYCD text,
    N_TORI_RYA_KANJI text,
    NOCD text,
    JIKB text,
    CHIK text,
    KIKI text,
    KISC text,
    SEIH text,
    KETE text,
    NOKI text,
    NOYM text,
    JYKN text,
    SGEN text,
    MITSU_SEIZO_GENKA text,
    KIKA_JUCHUJI_KIN text,
    KIKA_SEIZO_GENKA text,
    KIKA_HANB_SOGENK text,
    VAL_CHAIN_KBN_CD text,
    VAL_CHAIN_KBN_NM text,
    AFTER_BUZ_NM text,
    NAIG text,
    KOKUNAIGAI_KBN text
);

copy order_backlog from 'power_bi/order_backlog.txt'(delimiter '	', format csv, header true )
;

--変更納期用_受注時利益データ
drop table order_chg_del
;
create table order_chg_del(
    GOKI text,
    SHGP text,
    TODOFKN_CD text,
    JYCD text,
    NOCD text,
    NAIGAI_KBN text,
    JYGP text,
    NONYUSAKI_NM text,
    SEIHIN_NM text,
    CHIK text,
    SEIH text,
    SHUBETSU_CD text,
    NOKI text,
    HENKO_NOKI text,
    SHKAKAKEIKAKU text,
    JYKN text
);

copy order_chg_del from 'power_bi/order_chg_del.txt'(delimiter '	', format csv, header true )
;

drop table order_backlog_summary;
create table order_backlog_summary as(
 select
    ob.JUCHU_DATE ,
    ob.GOKI,
    ob.J_TORI_RYA_KANJI,
    ob.JYCD,
    ob.N_TORI_RYA_KANJI,
    ob.NOCD,
    ob.JIKB,
	jkt.jigyoumei,
    ob.CHIK,
    ob.KIKI,
    ob.KISC,
    ob.SEIH,
    ob.KETE,
    ob.NOKI,
    ob.NOYM,
    cast(to_number(ob.JYKN, 'FM999,999,999,999.000') as integer) as JYKN,
    ob.SGEN,
    ob.MITSU_SEIZO_GENKA,
    ob.KIKA_JUCHUJI_KIN,
    ob.KIKA_SEIZO_GENKA,
    ob.KIKA_HANB_SOGENK,
    ob.VAL_CHAIN_KBN_CD,
	substr(ob.VAL_CHAIN_KBN_CD, 1,2) as cat_cd,
    ob.VAL_CHAIN_KBN_NM,
    ob.AFTER_BUZ_NM,
    ob.NAIG,
    ob.KOKUNAIGAI_KBN,
    cast(ocd.HENKO_NOKI || '01' as date) as HENKO_NOKI,
	vkt.val_ktd,
	case 
	    when ocd.HENKO_NOKI < to_char(now(), 'yyyymm') then '過月度'
	    when ocd.HENKO_NOKI >= to_char(now(), 'yyyymm') then  to_char(cast(ocd.HENKO_NOKI || '01' as date), 'yy/mm')
	    else '21年度以降' end as past_judge,
	case
	    when ocd.HENKO_NOKI < to_char(now(), 'yyyymm') then '1'
	    when ocd.HENKO_NOKI >= to_char(now(), 'yyyymm') then '2'
	    else '3' end as time_order
	    

    from order_backlog as ob
	--受注残テーブルに変更納期テーブルを追加する
    left join order_chg_del as ocd
    on ob.goki = ocd.goki
	--バリューチェーン区分に活動系を割り振る
	left join valuechain_kubun_table as vkt
	on ob.VAL_CHAIN_KBN_CD = vkt.VAL_CHAIN_KBN_CD
	--事業区分コードに事業名を割り振る
	left join jigyou_kubun_table as jkt
	on ob.jikb = jkt.jigyou_kubun
	
	where 
	--カテゴリー区分を04,05,06,07にしぼる
	substr(ob.VAL_CHAIN_KBN_CD, 1,2) in ('04', '05', '06', '07')and
	--制御Kをのぞく
	ob.jikb <> 'K' and
	--外販のみにしぼる
	ob.naig = '1'
)
----------
