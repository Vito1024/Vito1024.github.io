# Linux Sort 的性能日记

> 为了加速import数据进mysql, 所以先对1亿条数据进行排序，本文记录下sort这一亿条数据的耗时情况

### 1. 数据类型
> 因为是mock数据， 所以数据可能针对bitcoin无效

| txid | vout | address | amount | confirmed | spent | block_height | block_hash |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0003hgjst9hh6n96u9otwwqu2minkxb36lsdchvakx4l1i1733zdbwm8t91u0v2c | 7  | bc03mdnuhwus1p2lpjj95gvj0bh51htlc1c4or2759r0kt8nhzl01c16r3a51n | 2.24293379 | true | false | 61288 | 0003hgjst9hh6n96u9otwwqu2minkxb36lsdchvakx4l1i1733zdbwm8t91u0v2c |
| 001p1285qo2lgnww7zqyqqy2xafl5a6yw48b7d2ja3hohi426vlwmgdvxp43yfth | 0 | bc03mdnuhwus1p2lpjj95gvj0bh51htlc1c4or2759r0kt8nhzl01c16r3a51n | 0.00000001 | true | false | 61288 | 0003hgjst9hh6n96u9otwwqu2minkxb36lsdchvakx4l1i1733zdbwm8t91u0v2c |
| 001woce1ww6yn4hi1auyd7uu9d0t4hr3kgrdf4drbjidsxe7sinsz5e436ta3uvg | 2 | bc03mdnuhwus1p2lpjj95gvj0bh51htlc1c4or2759r0kt8nhzl01c16r3a51n | 0.00000001 | true | false | 61288 | 001woce1ww6yn4hi1auyd7uu9d0t4hr3kgrdf4drbjidsxe7sinsz5e436ta3uvg |

这样的mock数据有1亿条

### 2. 排序命令
```bash
sort -t ',' -k3,3 -k1,1 -k2,2n mock_utxo_data.csv > sorted_utxo.csv
```
因为数据库主键为: PRIMARY KEY (address,txid, vout)，所以这里先根据address排序，然后根据txid排序，最后根据vout排序

### 3. 执行耗时
最后执行耗时大约为<b>1小时20分钟</b>。