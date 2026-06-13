# 世界杯临场赛程 / 赔率 / Polymarket 只读核验探针

用于 04:00 早盘、16:00 临场或 web 搜索不可用/不完整时，直接用公开 API 做赛程、赔率、Gamma 和 CLOB orderbook 核验。不要把某次工具不可用写成长期结论；本页记录的是可复用的只读核验路径。

## 1. 当前时间

```bash
date -u
TZ=Asia/Shanghai date
```

## 2. FIFA 赛程核验

按 UTC 窗口查询未来 24 小时；`idCompetition=17` 对应 FIFA World Cup。

```bash
python3 - <<'PY'
import urllib.request,json
u='https://api.fifa.com/api/v3/calendar/matches?from=2026-06-12T08:00:00Z&to=2026-06-13T08:00:00Z&idCompetition=17&language=en'
req=urllib.request.Request(u,headers={'User-Agent':'Mozilla/5.0','Accept':'application/json'})
data=json.load(urllib.request.urlopen(req,timeout=30))
for m in data.get('Results',[]):
    def loc(arr):
        return arr[0].get('Description') if isinstance(arr,list) and arr else arr
    print(m.get('IdMatch'), m.get('Date'), loc(m.get('StageName')), loc(m.get('GroupName')))
    print(' ', loc(m.get('Home',{}).get('TeamName')), 'vs', loc(m.get('Away',{}).get('TeamName')))
    print(' ', loc(m.get('Stadium',{}).get('Name')) if m.get('Stadium') else None)
PY
```

## 3. ESPN 赛程 + sportsbook 盘口

ESPN scoreboard 通常会返回赛程、场地、状态、DraftKings moneyline / spread / totals 快照；summary 可补 boxscore、rosters、news、odds，但首发/伤停不一定有。

```bash
python3 - <<'PY'
import urllib.request,json
headers={'User-Agent':'Mozilla/5.0','Accept':'application/json'}
for date in ['20260612','20260613']:
    u=f'https://site.api.espn.com/apis/site/v2/sports/soccer/fifa.world/scoreboard?dates={date}'
    data=json.load(urllib.request.urlopen(urllib.request.Request(u,headers=headers),timeout=20))
    print('\nDATE',date)
    for e in data.get('events',[]):
        comp=e['competitions'][0]
        print(e.get('id'), e.get('name'), e.get('date'), e.get('status',{}).get('type',{}).get('description'))
        print(' venue:', comp.get('venue',{}).get('fullName'), comp.get('venue',{}).get('address',{}).get('city'))
        if comp.get('odds'):
            o=comp['odds'][0]
            print(' odds:', o.get('details'), 'OU', o.get('overUnder'), 'home/away/draw ML:',
                  o.get('moneyline',{}).get('home'), o.get('moneyline',{}).get('away'), o.get('moneyline',{}).get('draw'))
PY
```

若需要确认是否有首发/伤停：

```bash
python3 - <<'PY'
import urllib.request,json
headers={'User-Agent':'Mozilla/5.0','Accept':'application/json'}
for event_id in ['760416','760417']:
    u=f'https://site.api.espn.com/apis/site/v2/sports/soccer/fifa.world/summary?event={event_id}'
    data=json.load(urllib.request.urlopen(urllib.request.Request(u,headers=headers),timeout=20))
    print(event_id, data.keys())
    print('status:', data.get('header',{}).get('competitions',[{}])[0].get('status'))
    print('has injuries key:', 'injuries' in data)
    print('boxscore keys:', data.get('boxscore',{}).keys() if isinstance(data.get('boxscore'),dict) else None)
PY
```

如果没有明确首发/伤停字段，不要编造；输出“首发/伤停待确认 / 暂无可靠实时数据”，并降低或保持保守仓位。

## 4. Polymarket Gamma 世界杯 event / child markets

普通 `events?series_id=11433` 只覆盖主胜平负；`events/keyset?...include_best_lines=true` 能取到 More Markets、O/U、半场、球员 props 等子事件。

```bash
python3 - <<'PY'
import urllib.request,json
headers={'User-Agent':'Mozilla/5.0','Accept':'application/json','Origin':'https://polymarket.com','Referer':'https://polymarket.com/'}
u='https://gamma-api.polymarket.com/events/keyset?tag_id=102232&series_id=11433&closed=false&order=startTime&ascending=true&limit=100&include_best_lines=true'
data=json.load(urllib.request.urlopen(urllib.request.Request(u,headers=headers),timeout=30))
for ev in data.get('events',[]):
    if '2026-06-12' in ev.get('slug',''):
        print(ev.get('id'), ev.get('slug'), ev.get('title'), 'markets', len(ev.get('markets',[])))
PY
```

主胜平负可用：

```bash
https://gamma-api.polymarket.com/events?closed=false&limit=100&series_id=11433
```

大小球通常在 slug `*-more-markets` 里，市场题名形如 `Team A vs. Team B: O/U 2.5`，`outcomes=["Over","Under"]` 时 Under token 为 `clobTokenIds[1]`，但仍以实际 outcomes 顺序为准。

## 5. CLOB orderbook 核验

拿到 `clobTokenIds` 后必须查 orderbook，确认 token 有效且有盘口。

```bash
python3 - <<'PY'
import urllib.request,json
TOKENS={'example':'<clobTokenId>'}
headers={'User-Agent':'Mozilla/5.0','Accept':'application/json'}
for name,t in TOKENS.items():
    u='https://clob.polymarket.com/book?token_id='+t
    data=json.load(urllib.request.urlopen(urllib.request.Request(u,headers=headers),timeout=20))
    def p(x):
        try: return float(x.get('price'))
        except Exception: return None
    bids=data.get('bids',[]); asks=data.get('asks',[])
    best_bid=max([p(x) for x in bids if p(x) is not None], default=None)
    best_ask=min([p(x) for x in asks if p(x) is not None], default=None)
    print(name, 'best_bid/best_ask', best_bid, best_ask, 'bids_n', len(bids), 'asks_n', len(asks))
PY
```

## 6. 输出注意

- 若没有早盘原始卡片，不要硬说“上调/降级”的历史原因；写“未读取到早盘原始卡片，按临场重新核验后给出调整口径”。
- Polymarket 价格与 sportsbook 盘口方向一致但价格已偏热时，仓位要降级，给价格阈值而不是机械 1 份。
- 只读报告不自动下单；实盘仍必须走 HK 服务器、余额/allowance、最新 token 和 orderbook 安全门。
