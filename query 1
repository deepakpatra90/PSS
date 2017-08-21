  WITH booking_date AS
    (SELECT min_res_book_dt,
      max_res_book_dt,
      ROUND(max_res_book_dt - min_res_book_dt) diff
    FROM
      (SELECT TRUNC(MIN(to_date('29-JUL-2017 00:00:00','DD-MON-YYYY HH24:MI:SS'))) min_res_book_dt,
        TRUNC(MAX(to_date('30-JUL-2017 00:00:00','DD-MON-YYYY HH24:MI:SS'))) + 86399 / 86400 max_res_book_dt
      FROM dual)
                ),  
    row_every_5_mins AS
    (SELECT TRUNC(min_res_book_dt) + (rownum-1)*5/1440 t_from,
      TRUNC(min_res_book_dt)       + rownum*5/1440 t_to
    FROM booking_date
      CONNECT BY level <= (1440/5) * diff
    ),
    row_channel AS
    (SELECT 'GDS' res_channel FROM dual
    UNION
    SELECT 'STANDARD' res_channel FROM dual
    UNION
    SELECT 'TPAPI' res_channel FROM dual
    ),
    row_result AS
    (SELECT * FROM row_every_5_mins r, row_channel c
    ),
    
  final_res AS
  (SELECT t1.res_channel,    
          lpad(to_number(to_char(t1.t_from,'HH24MI')),4,0) t_from,
          lpad(to_number(to_char(t1.t_to,'HH24MI')),4,0) t_to,
  
    NVL(t2.booking_count,0) booking_count
  FROM row_result t1
  LEFT OUTER JOIN
    (SELECT ch.res_channel,
      COUNT(sd.confirmation_num) booking_count,
      r.t_From,
      r.t_to
  
    FROM RESERVATION_segs sd ,
      res_channels ch ,
      row_every_5_mins r
    WHERE ch.res_channel_id(+) = sd.res_channel_id--sd.res_type
    AND ch.res_channel(+)        IN ('GDS','STANDARD','TPAPI')
    AND sd.book_date(+)        >= r.t_From
    AND sd.book_date(+)         < r.t_to
    GROUP BY r.t_From,
      r.t_to,
      ch.res_channel
    ORDER BY t_From) t2  
  
  ON t1.t_From        = t2.t_From
  AND t1.t_to         = t2.t_to
  AND (t1.res_channel = t2.res_channel
  OR t2.res_channel  IS NULL)
  ORDER BY t1.t_From,t1.t_to)
  
  select 
  distinct rs.RES_CHANNEL, 
  rs.T_FROM, 
  rs.T_TO, 
  rs.BOOKING_COUNT,
  tb.min_count,
  (CASE WHEN rs.BOOKING_COUNT < tb.min_count then 'YES' else 'NO' END)  "MSG"
  from
  final_res rs
  INNER JOIN  TAB_BKD_CHK tb
  on RS.RES_CHANNEL = tb.res_type
  and RS.T_FROM = tb.start_time
  and RS.T_TO = tb.end_time
  where RS.RES_CHANNEL=NVL('',RS.RES_CHANNEL);
  
  
