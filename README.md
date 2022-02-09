# ETL with Pandas
---

*  Data is extracted using the Binance API.


* Pandas will be used to transform and normalize the json data obtained from Binance


* For convenience, data will be loaded into a SQLITE DB

## Extraction


```python
import requests

# Set the request parameters
url = 'https://api.binance.com/api/v3/exchangeInfo'

# Do the HTTP get request
response = requests.get(url)

# Check for HTTP codes other than 200
if response.status_code != 200:
    print('Status:', response.status_code, 'Problem with the request. Exiting.')
    exit()
else:
    print('Success')
```

    Success



```python
import pandas as pd
pd.set_option('display.max_rows', 50000)
pd.set_option('display.max_columns', 50000)
#pd.set_option('display.max_colwidth', None)

# Generate Dict from json response 
data = response.json()

# Create df from data
df = pd.DataFrame(list(data.items())) 

# Preview
df.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>timezone</td>
      <td>UTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>serverTime</td>
      <td>1644436739538</td>
    </tr>
    <tr>
      <th>2</th>
      <td>rateLimits</td>
      <td>[{'rateLimitType': 'REQUEST_WEIGHT', 'interval...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>exchangeFilters</td>
      <td>[]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>symbols</td>
      <td>[{'symbol': 'ETHBTC', 'status': 'TRADING', 'ba...</td>
    </tr>
  </tbody>
</table>
</div>




```python
rateLimits = pd.DataFrame(data['rateLimits'])
rateLimits
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rateLimitType</th>
      <th>interval</th>
      <th>intervalNum</th>
      <th>limit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>REQUEST_WEIGHT</td>
      <td>MINUTE</td>
      <td>1</td>
      <td>1200</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ORDERS</td>
      <td>SECOND</td>
      <td>10</td>
      <td>50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ORDERS</td>
      <td>DAY</td>
      <td>1</td>
      <td>160000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>RAW_REQUESTS</td>
      <td>MINUTE</td>
      <td>5</td>
      <td>6100</td>
    </tr>
  </tbody>
</table>
</div>




```python
symbols = pd.DataFrame(data['symbols'])
display(symbols.info())
symbols.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1932 entries, 0 to 1931
    Data columns (total 17 columns):
     #   Column                      Non-Null Count  Dtype 
    ---  ------                      --------------  ----- 
     0   symbol                      1932 non-null   object
     1   status                      1932 non-null   object
     2   baseAsset                   1932 non-null   object
     3   baseAssetPrecision          1932 non-null   int64 
     4   quoteAsset                  1932 non-null   object
     5   quotePrecision              1932 non-null   int64 
     6   quoteAssetPrecision         1932 non-null   int64 
     7   baseCommissionPrecision     1932 non-null   int64 
     8   quoteCommissionPrecision    1932 non-null   int64 
     9   orderTypes                  1932 non-null   object
     10  icebergAllowed              1932 non-null   bool  
     11  ocoAllowed                  1932 non-null   bool  
     12  quoteOrderQtyMarketAllowed  1932 non-null   bool  
     13  isSpotTradingAllowed        1932 non-null   bool  
     14  isMarginTradingAllowed      1932 non-null   bool  
     15  filters                     1932 non-null   object
     16  permissions                 1932 non-null   object
    dtypes: bool(5), int64(5), object(7)
    memory usage: 190.7+ KB



    None





<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>symbol</th>
      <th>status</th>
      <th>baseAsset</th>
      <th>baseAssetPrecision</th>
      <th>quoteAsset</th>
      <th>quotePrecision</th>
      <th>quoteAssetPrecision</th>
      <th>baseCommissionPrecision</th>
      <th>quoteCommissionPrecision</th>
      <th>orderTypes</th>
      <th>icebergAllowed</th>
      <th>ocoAllowed</th>
      <th>quoteOrderQtyMarketAllowed</th>
      <th>isSpotTradingAllowed</th>
      <th>isMarginTradingAllowed</th>
      <th>filters</th>
      <th>permissions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ETHBTC</td>
      <td>TRADING</td>
      <td>ETH</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>[LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, ...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>[SPOT, MARGIN]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LTCBTC</td>
      <td>TRADING</td>
      <td>LTC</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>[LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, ...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>[SPOT, MARGIN]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BNBBTC</td>
      <td>TRADING</td>
      <td>BNB</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>[LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, ...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>[SPOT, MARGIN]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NEOBTC</td>
      <td>TRADING</td>
      <td>NEO</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>[LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, ...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>[SPOT, MARGIN]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>QTUMETH</td>
      <td>TRADING</td>
      <td>QTUM</td>
      <td>8</td>
      <td>ETH</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>[LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, ...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>[SPOT]</td>
    </tr>
  </tbody>
</table>
</div>



## Transformation


```python
columns = list(symbols.columns)
for i in columns:
    try:
        print(i)
        print(symbols[i].unique())
    except Exception as e:
        print('!!!!!!!!!!!!!!!!!!!')
        print(e)
        print('!!!!!!!!!!!!!!!!!!!')
        next
```

    symbol
    ['ETHBTC' 'LTCBTC' 'BNBBTC' ... 'WOOBNB' 'WOOBUSD' 'WOOUSDT']
    status
    ['TRADING' 'BREAK']
    baseAsset
    ['ETH' 'LTC' 'BNB' 'NEO' 'QTUM' 'EOS' 'SNT' 'BNT' 'BCC' 'GAS' 'BTC' 'HSR'
     'OAX' 'DNT' 'MCO' 'ICN' 'WTC' 'LRC' 'YOYO' 'OMG' 'ZRX' 'STRAT' 'SNGLS'
     'BQX' 'KNC' 'FUN' 'SNM' 'IOTA' 'LINK' 'XVG' 'SALT' 'MDA' 'MTL' 'SUB'
     'ETC' 'MTH' 'ENG' 'ZEC' 'AST' 'DASH' 'BTG' 'EVX' 'REQ' 'VIB' 'TRX' 'POWR'
     'ARK' 'XRP' 'MOD' 'ENJ' 'STORJ' 'VEN' 'KMD' 'NULS' 'RCN' 'RDN' 'XMR'
     'DLT' 'AMB' 'BAT' 'BCPT' 'ARN' 'GVT' 'CDT' 'GXS' 'POE' 'QSP' 'BTS' 'XZC'
     'LSK' 'TNT' 'FUEL' 'MANA' 'BCD' 'DGD' 'ADX' 'ADA' 'PPT' 'CMT' 'XLM' 'CND'
     'LEND' 'WABI' 'TNB' 'WAVES' 'GTO' 'ICX' 'OST' 'ELF' 'AION' 'NEBL' 'BRD'
     'EDO' 'WINGS' 'NAV' 'LUN' 'TRIG' 'APPC' 'VIBE' 'RLC' 'INS' 'PIVX' 'IOST'
     'CHAT' 'STEEM' 'NANO' 'VIA' 'BLZ' 'AE' 'RPX' 'NCASH' 'POA' 'ZIL' 'ONT'
     'STORM' 'XEM' 'WAN' 'WPR' 'QLC' 'SYS' 'GRS' 'CLOAK' 'GNT' 'LOOM' 'BCN'
     'REP' 'TUSD' 'ZEN' 'SKY' 'CVC' 'THETA' 'IOTX' 'QKC' 'AGI' 'NXS' 'DATA'
     'SC' 'NPXS' 'KEY' 'NAS' 'MFT' 'DENT' 'ARDR' 'HOT' 'VET' 'DOCK' 'POLY'
     'PHX' 'HC' 'GO' 'PAX' 'RVN' 'DCR' 'USDC' 'MITH' 'BCHABC' 'BCHSV' 'REN'
     'BTT' 'USDS' 'ONG' 'FET' 'CELR' 'MATIC' 'ATOM' 'PHB' 'TFUEL' 'ONE' 'FTM'
     'BTCB' 'ALGO' 'USDSB' 'ERD' 'DOGE' 'DUSK' 'BGBP' 'ANKR' 'WIN' 'COS'
     'TUSDB' 'COCOS' 'TOMO' 'PERL' 'CHZ' 'BAND' 'BUSD' 'BEAM' 'XTZ' 'HBAR'
     'NKN' 'STX' 'KAVA' 'ARPA' 'CTXC' 'BCH' 'TROY' 'VITE' 'FTT' 'USDT' 'EUR'
     'OGN' 'DREP' 'BULL' 'BEAR' 'ETHBULL' 'ETHBEAR' 'TCT' 'WRX' 'LTO'
     'EOSBULL' 'EOSBEAR' 'XRPBULL' 'XRPBEAR' 'MBL' 'COTI' 'BNBBULL' 'BNBBEAR'
     'STPT' 'SOL' 'CTSI' 'HIVE' 'CHR' 'BTCUP' 'BTCDOWN' 'MDT' 'STMX' 'IQ'
     'PNT' 'GBP' 'DGB' 'COMP' 'BKRW' 'SXP' 'SNX' 'ETHUP' 'ETHDOWN' 'ADAUP'
     'ADADOWN' 'LINKUP' 'LINKDOWN' 'VTHO' 'IRIS' 'MKR' 'DAI' 'RUNE' 'AUD'
     'FIO' 'BNBUP' 'BNBDOWN' 'XTZUP' 'XTZDOWN' 'AVA' 'BAL' 'YFI' 'JST' 'SRM'
     'ANT' 'CRV' 'SAND' 'OCEAN' 'NMR' 'DOT' 'LUNA' 'IDEX' 'RSR' 'PAXG' 'WNXM'
     'TRB' 'BZRX' 'WBTC' 'SUSHI' 'YFII' 'KSM' 'EGLD' 'DIA' 'UMA' 'EOSUP'
     'EOSDOWN' 'TRXUP' 'TRXDOWN' 'XRPUP' 'XRPDOWN' 'DOTUP' 'DOTDOWN' 'BEL'
     'WING' 'SWRV' 'LTCUP' 'LTCDOWN' 'CREAM' 'UNI' 'NBS' 'OXT' 'SUN' 'AVAX'
     'HNT' 'BAKE' 'BURGER' 'FLM' 'SCRT' 'CAKE' 'SPARTA' 'UNIUP' 'UNIDOWN'
     'ORN' 'UTK' 'XVS' 'ALPHA' 'VIDT' 'AAVE' 'NEAR' 'SXPUP' 'SXPDOWN' 'FIL'
     'FILUP' 'FILDOWN' 'YFIUP' 'YFIDOWN' 'INJ' 'AERGO' 'EASY' 'AUDIO' 'CTK'
     'BCHUP' 'BCHDOWN' 'BOT' 'AKRO' 'KP3R' 'AXS' 'HARD' 'RENBTC' 'SLP' 'CVP'
     'STRAX' 'FOR' 'UNFI' 'FRONT' 'BCHA' 'ROSE' 'HEGIC' 'AAVEUP' 'AAVEDOWN'
     'PROM' 'SKL' 'SUSD' 'COVER' 'GLM' 'GHST' 'SUSHIUP' 'SUSHIDOWN' 'XLMUP'
     'XLMDOWN' 'DF' 'GRT' 'JUV' 'PSG' '1INCH' 'REEF' 'OG' 'ATM' 'ASR' 'CELO'
     'RIF' 'BTCST' 'TRU' 'DEXE' 'CKB' 'TWT' 'FIRO' 'BETH' 'PROS' 'LIT' 'SFP'
     'FXS' 'DODO' 'UFT' 'ACM' 'AUCTION' 'PHA' 'TVK' 'BADGER' 'FIS' 'OM' 'POND'
     'DEGO' 'ALICE' 'BIFI' 'LINA' 'PERP' 'RAMP' 'SUPER' 'CFX' 'EPS' 'AUTO'
     'TKO' 'PUNDIX' 'TLM' '1INCHUP' '1INCHDOWN' 'MIR' 'BAR' 'FORTH' 'EZ'
     'SHIB' 'ICP' 'AR' 'POLS' 'MDX' 'MASK' 'LPT' 'AGIX' 'NU' 'ATA' 'GTC'
     'TORN' 'KEEP' 'ERN' 'KLAY' 'BOND' 'MLN' 'QUICK' 'C98' 'CLV' 'QNT' 'FLOW'
     'XEC' 'MINA' 'RAY' 'FARM' 'ALPACA' 'MBOX' 'VGX' 'WAXP' 'TRIBE' 'GNO'
     'DYDX' 'USDP' 'GALA' 'ILV' 'YGG' 'FIDA' 'AGLD' 'RAD' 'BETA' 'RARE' 'SSV'
     'LAZIO' 'CHESS' 'DAR' 'BNX' 'RGT' 'MOVR' 'CITY' 'ENS' 'QI' 'PORTO'
     'JASMY' 'AMP' 'PLA' 'PYR' 'RNDR' 'ALCX' 'SANTOS' 'MC' 'ANY' 'BICO' 'FLUX'
     'VOXEL' 'HIGH' 'CVX' 'PEOPLE' 'OOKI' 'SPELL' 'UST' 'JOE' 'ACH' 'IMX'
     'GLMR' 'LOKA' 'API3' 'BTTC' 'ACA' 'ANC' 'BDOT' 'XNO' 'WOO']
    baseAssetPrecision
    [8 2 1]
    quoteAsset
    ['BTC' 'ETH' 'USDT' 'BNB' 'TUSD' 'PAX' 'USDC' 'XRP' 'USDS' 'TRX' 'BUSD'
     'NGN' 'RUB' 'TRY' 'EUR' 'ZAR' 'BKRW' 'IDRT' 'GBP' 'UAH' 'BIDR' 'AUD'
     'DAI' 'BRL' 'BVND' 'VAI' 'GYEN' 'USDP' 'DOGE' 'UST' 'DOT']
    quotePrecision
    [8 2]
    quoteAssetPrecision
    [8 2]
    baseCommissionPrecision
    [8 2 1]
    quoteCommissionPrecision
    [8 2]
    orderTypes
    !!!!!!!!!!!!!!!!!!!
    unhashable type: 'list'
    !!!!!!!!!!!!!!!!!!!
    icebergAllowed
    [ True]
    ocoAllowed
    [ True]
    quoteOrderQtyMarketAllowed
    [ True False]
    isSpotTradingAllowed
    [ True False]
    isMarginTradingAllowed
    [ True False]
    filters
    !!!!!!!!!!!!!!!!!!!
    unhashable type: 'list'
    !!!!!!!!!!!!!!!!!!!
    permissions
    !!!!!!!!!!!!!!!!!!!
    unhashable type: 'list'
    !!!!!!!!!!!!!!!!!!!



```python
symbols = pd.DataFrame(data['symbols'])
symbols['orderTypes'] = symbols['orderTypes'].apply(', '.join)
symbols['permissions'] = symbols['permissions'].apply(', '.join)

display(symbols.info())
symbols.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1932 entries, 0 to 1931
    Data columns (total 17 columns):
     #   Column                      Non-Null Count  Dtype 
    ---  ------                      --------------  ----- 
     0   symbol                      1932 non-null   object
     1   status                      1932 non-null   object
     2   baseAsset                   1932 non-null   object
     3   baseAssetPrecision          1932 non-null   int64 
     4   quoteAsset                  1932 non-null   object
     5   quotePrecision              1932 non-null   int64 
     6   quoteAssetPrecision         1932 non-null   int64 
     7   baseCommissionPrecision     1932 non-null   int64 
     8   quoteCommissionPrecision    1932 non-null   int64 
     9   orderTypes                  1932 non-null   object
     10  icebergAllowed              1932 non-null   bool  
     11  ocoAllowed                  1932 non-null   bool  
     12  quoteOrderQtyMarketAllowed  1932 non-null   bool  
     13  isSpotTradingAllowed        1932 non-null   bool  
     14  isMarginTradingAllowed      1932 non-null   bool  
     15  filters                     1932 non-null   object
     16  permissions                 1932 non-null   object
    dtypes: bool(5), int64(5), object(7)
    memory usage: 190.7+ KB



    None





<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>symbol</th>
      <th>status</th>
      <th>baseAsset</th>
      <th>baseAssetPrecision</th>
      <th>quoteAsset</th>
      <th>quotePrecision</th>
      <th>quoteAssetPrecision</th>
      <th>baseCommissionPrecision</th>
      <th>quoteCommissionPrecision</th>
      <th>orderTypes</th>
      <th>icebergAllowed</th>
      <th>ocoAllowed</th>
      <th>quoteOrderQtyMarketAllowed</th>
      <th>isSpotTradingAllowed</th>
      <th>isMarginTradingAllowed</th>
      <th>filters</th>
      <th>permissions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ETHBTC</td>
      <td>TRADING</td>
      <td>ETH</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LTCBTC</td>
      <td>TRADING</td>
      <td>LTC</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BNBBTC</td>
      <td>TRADING</td>
      <td>BNB</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NEOBTC</td>
      <td>TRADING</td>
      <td>NEO</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>QTUMETH</td>
      <td>TRADING</td>
      <td>QTUM</td>
      <td>8</td>
      <td>ETH</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>[{'filterType': 'PRICE_FILTER', 'minPrice': '0...</td>
      <td>SPOT</td>
    </tr>
  </tbody>
</table>
</div>




```python
price_filter = pd.DataFrame()
percent_price = pd.DataFrame()
lot_size = pd.DataFrame()
min_notional = pd.DataFrame()
iceberg_parts = pd.DataFrame()
market_lot_size = pd.DataFrame()
max_num_orders = pd.DataFrame()
max_num_algo_orders = pd.DataFrame()


i = 0

while i < len(symbols['filters']):
    
    j = 0
    
    while j < len(symbols['filters'][i]):
        check1 = pd.DataFrame(symbols['filters'][i][j],index=symbols['filters'][i][j].keys()).reset_index(drop=True).drop_duplicates(keep='first')
        
        check1['symbol_key'] = symbols['symbol'][i]
        
        if symbols['filters'][i][j]['filterType'] == 'PRICE_FILTER':
            price_filter = price_filter.append(check1)
            
        elif symbols['filters'][i][j]['filterType'] == 'PERCENT_PRICE':
            percent_price = percent_price.append(check1)
            
        elif symbols['filters'][i][j]['filterType'] == 'LOT_SIZE':
            lot_size = lot_size.append(check1)

        elif symbols['filters'][i][j]['filterType'] == 'MIN_NOTIONAL':
            min_notional = min_notional.append(check1)
            
        elif symbols['filters'][i][j]['filterType'] == 'ICEBERG_PARTS':
            iceberg_parts = iceberg_parts.append(check1)
            
        elif symbols['filters'][i][j]['filterType'] == 'MARKET_LOT_SIZE':
            market_lot_size = market_lot_size.append(check1)
            
        elif symbols['filters'][i][j]['filterType'] == 'MAX_NUM_ORDERS':
            max_num_orders = max_num_orders.append(check1)            
            
        elif symbols['filters'][i][j]['filterType'] == 'MAX_NUM_ALGO_ORDERS':
            max_num_algo_orders = max_num_algo_orders.append(check1)
            
        else:
            pass
            
        j = j+1
    
    i = i+1
```


```python
price_filter = price_filter.reset_index(drop=True)
percent_price = percent_price.reset_index(drop=True)
lot_size = lot_size.reset_index(drop=True)
min_notional = min_notional.reset_index(drop=True)
iceberg_parts = iceberg_parts.reset_index(drop=True)
market_lot_size = market_lot_size.reset_index(drop=True)
max_num_orders = max_num_orders.reset_index(drop=True)
max_num_algo_orders = max_num_algo_orders.reset_index(drop=True)
```


```python
display(price_filter.head())
display(percent_price.head())
display(lot_size.head())
display(min_notional.head())
display(iceberg_parts.head())
display(market_lot_size.head())
display(max_num_orders.head())
display(max_num_algo_orders.head())
```


<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>minPrice</th>
      <th>maxPrice</th>
      <th>tickSize</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>PRICE_FILTER</td>
      <td>0.00000100</td>
      <td>922327.00000000</td>
      <td>0.00000100</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PRICE_FILTER</td>
      <td>0.00000100</td>
      <td>100000.00000000</td>
      <td>0.00000100</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PRICE_FILTER</td>
      <td>0.00000100</td>
      <td>100000.00000000</td>
      <td>0.00000100</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PRICE_FILTER</td>
      <td>0.00000100</td>
      <td>100000.00000000</td>
      <td>0.00000100</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PRICE_FILTER</td>
      <td>0.00000100</td>
      <td>1000.00000000</td>
      <td>0.00000100</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>multiplierUp</th>
      <th>multiplierDown</th>
      <th>avgPriceMins</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>PERCENT_PRICE</td>
      <td>5</td>
      <td>0.2</td>
      <td>5</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PERCENT_PRICE</td>
      <td>5</td>
      <td>0.2</td>
      <td>5</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PERCENT_PRICE</td>
      <td>5</td>
      <td>0.2</td>
      <td>5</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PERCENT_PRICE</td>
      <td>5</td>
      <td>0.2</td>
      <td>5</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PERCENT_PRICE</td>
      <td>5</td>
      <td>0.2</td>
      <td>5</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>minQty</th>
      <th>maxQty</th>
      <th>stepSize</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LOT_SIZE</td>
      <td>0.00010000</td>
      <td>100000.00000000</td>
      <td>0.00010000</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LOT_SIZE</td>
      <td>0.00100000</td>
      <td>100000.00000000</td>
      <td>0.00100000</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>LOT_SIZE</td>
      <td>0.00100000</td>
      <td>100000.00000000</td>
      <td>0.00100000</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LOT_SIZE</td>
      <td>0.01000000</td>
      <td>100000.00000000</td>
      <td>0.01000000</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LOT_SIZE</td>
      <td>0.10000000</td>
      <td>90000000.00000000</td>
      <td>0.10000000</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>minNotional</th>
      <th>applyToMarket</th>
      <th>avgPriceMins</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MIN_NOTIONAL</td>
      <td>0.00010000</td>
      <td>True</td>
      <td>5</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MIN_NOTIONAL</td>
      <td>0.00010000</td>
      <td>True</td>
      <td>5</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MIN_NOTIONAL</td>
      <td>0.00010000</td>
      <td>True</td>
      <td>5</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MIN_NOTIONAL</td>
      <td>0.00010000</td>
      <td>True</td>
      <td>5</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MIN_NOTIONAL</td>
      <td>0.00500000</td>
      <td>True</td>
      <td>5</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>limit</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ICEBERG_PARTS</td>
      <td>10</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ICEBERG_PARTS</td>
      <td>10</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ICEBERG_PARTS</td>
      <td>10</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ICEBERG_PARTS</td>
      <td>10</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ICEBERG_PARTS</td>
      <td>10</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>minQty</th>
      <th>maxQty</th>
      <th>stepSize</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MARKET_LOT_SIZE</td>
      <td>0.00000000</td>
      <td>918.14426441</td>
      <td>0.00000000</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MARKET_LOT_SIZE</td>
      <td>0.00000000</td>
      <td>8434.74222029</td>
      <td>0.00000000</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MARKET_LOT_SIZE</td>
      <td>0.00000000</td>
      <td>12094.66837317</td>
      <td>0.00000000</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MARKET_LOT_SIZE</td>
      <td>0.00000000</td>
      <td>6967.66238359</td>
      <td>0.00000000</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MARKET_LOT_SIZE</td>
      <td>0.00000000</td>
      <td>5919.22145833</td>
      <td>0.00000000</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>maxNumOrders</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MAX_NUM_ORDERS</td>
      <td>200</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MAX_NUM_ORDERS</td>
      <td>200</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MAX_NUM_ORDERS</td>
      <td>200</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MAX_NUM_ORDERS</td>
      <td>200</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MAX_NUM_ORDERS</td>
      <td>200</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filterType</th>
      <th>maxNumAlgoOrders</th>
      <th>symbol_key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MAX_NUM_ALGO_ORDERS</td>
      <td>5</td>
      <td>ETHBTC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MAX_NUM_ALGO_ORDERS</td>
      <td>5</td>
      <td>LTCBTC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MAX_NUM_ALGO_ORDERS</td>
      <td>5</td>
      <td>BNBBTC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MAX_NUM_ALGO_ORDERS</td>
      <td>5</td>
      <td>NEOBTC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MAX_NUM_ALGO_ORDERS</td>
      <td>5</td>
      <td>QTUMETH</td>
    </tr>
  </tbody>
</table>
</div>



```python
symbols.drop('filters', axis=1, inplace=True)
symbols.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>symbol</th>
      <th>status</th>
      <th>baseAsset</th>
      <th>baseAssetPrecision</th>
      <th>quoteAsset</th>
      <th>quotePrecision</th>
      <th>quoteAssetPrecision</th>
      <th>baseCommissionPrecision</th>
      <th>quoteCommissionPrecision</th>
      <th>orderTypes</th>
      <th>icebergAllowed</th>
      <th>ocoAllowed</th>
      <th>quoteOrderQtyMarketAllowed</th>
      <th>isSpotTradingAllowed</th>
      <th>isMarginTradingAllowed</th>
      <th>permissions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ETHBTC</td>
      <td>TRADING</td>
      <td>ETH</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LTCBTC</td>
      <td>TRADING</td>
      <td>LTC</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BNBBTC</td>
      <td>TRADING</td>
      <td>BNB</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NEOBTC</td>
      <td>TRADING</td>
      <td>NEO</td>
      <td>8</td>
      <td>BTC</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>SPOT, MARGIN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>QTUMETH</td>
      <td>TRADING</td>
      <td>QTUM</td>
      <td>8</td>
      <td>ETH</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>LIMIT, LIMIT_MAKER, MARKET, STOP_LOSS_LIMIT, T...</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>SPOT</td>
    </tr>
  </tbody>
</table>
</div>



## Load


```python
import os
import sqlite3

path = os.getcwd()

try:
    conn = sqlite3.connect(path+"/sqlite.db")
    print('Opened Database Successfully')
except Exception as e:
    print(e)
```

    Opened Database Successfully



```python
try:
    cur = conn.cursor()
    print("Cursor Object Created")
except Exception as e:
    print(e)    
```

    Cursor Object Created



```python
try:
    cur.executescript('''
            CREATE TABLE symbols 
            (
              symbol VARCHAR(50) PRIMARY KEY NOT NULL 
            , status VARCHAR(50) NOT NULL 
            , baseAsset VARCHAR(50) NOT NULL
            , baseAssetPrecision INT NOT NULL
            , quoteAsset VARCHAR(50) NOT NULL
            , quotePrecision INT NOT NULL
            , quoteAssetPrecision INT NOT NULL
            , baseCommissionPrecision INT NOT NULL
            , quoteCommissionPrecision INT NOT NULL
            , orderTypes VARCHAR(100) NOT NULL
            , icebergAllowed BOOL
            , ocoAllowed BOOL
            , quoteOrderQtyMarketAllowed BOOL
            , isSpotTradingAllowed BOOL
            , isMarginTradingAllowed BOOL
            , permissions VARCHAR(100) NOT NULL
            );
            
            CREATE TABLE price_filter 
            (
              filterType VARCHAR(12) NOT NULL 
            , minPrice INT NOT NULL 
            , maxPrice INT NOT NULL
            , tickSize INT NOT NULL
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE percent_price
            (
              filterType VARCHAR(13) NOT NULL 
            , multiplierUp INT NOT NULL 
            , multiplierDown INT NOT NULL
            , avgPriceMins INT NOT NULL
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE lot_size 
            (
              filterType VARCHAR(12) NOT NULL 
            , minQty INT NOT NULL 
            , maxQty INT NOT NULL
            , stepSize INT NOT NULL
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE min_notional 
            (
              filterType VARCHAR(8) NOT NULL 
            , minNotional INT NOT NULL 
            , applyToMarket BOOL
            , avgPriceMins INT NOT NULL
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );

            CREATE TABLE iceberg_parts 
            (
              filterType VARCHAR(13) NOT NULL 
            , limit_ INT NOT NULL 
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE market_lot_size 
            (
              filterType VARCHAR(15) NOT NULL 
            , minQty INT NOT NULL 
            , maxQty INT NOT NULL
            , stepSize INT NOT NULL
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE max_num_orders 
            (
              filterType VARCHAR(14) NOT NULL 
            , maxNumOrders INT NOT NULL 
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            CREATE TABLE max_num_algo_orders 
            (
              filterType VARCHAR(19) NOT NULL 
            , maxNumAlgoOrders INT NOT NULL 
            , symbol_key VARCHAR(50) NOT NULL 
            , FOREIGN KEY (symbol_key)
                           REFERENCES symbols (symbol)
            );
            
            ''')
    
    print('Tables created successfully')

except Exception as e:
    print(e)  
```

    Tables created successfully



```python
symbols.to_sql('symbols',con=conn,if_exists='append',index=False)
price_filter.to_sql('price_filter',con=conn,if_exists='append',index=False)
percent_price.to_sql('percent_price',con=conn,if_exists='append',index=False)
lot_size.to_sql('lot_size',con=conn,if_exists='append',index=False)
min_notional.to_sql('min_notional',con=conn,if_exists='append',index=False)
iceberg_parts.to_sql('iceberg_parts',con=conn,if_exists='append',index=False)
market_lot_size.to_sql('market_lot_size',con=conn,if_exists='append',index=False)
max_num_orders.to_sql('max_num_orders',con=conn,if_exists='append',index=False)
max_num_algo_orders.to_sql('max_num_algo_orders',con=conn,if_exists='append',index=False)
```


```python
display(pd.read_sql('''pragma table_info(symbols);''',conn))
display(pd.read_sql('''pragma table_info(price_filter);''',conn))
display(pd.read_sql('''pragma table_info(percent_price);''',conn))
display(pd.read_sql('''pragma table_info(lot_size);''',conn))
display(pd.read_sql('''pragma table_info(min_notional);''',conn))
display(pd.read_sql('''pragma table_info(iceberg_parts);''',conn))
display(pd.read_sql('''pragma table_info(market_lot_size);''',conn))
display(pd.read_sql('''pragma table_info(max_num_orders);''',conn))
display(pd.read_sql('''pragma table_info(max_num_algo_orders);''',conn))
```


<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>symbol</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>status</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>baseAsset</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>baseAssetPrecision</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>quoteAsset</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>quotePrecision</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>quoteAssetPrecision</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>baseCommissionPrecision</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>quoteCommissionPrecision</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>orderTypes</td>
      <td>VARCHAR(100)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>icebergAllowed</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>ocoAllowed</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>quoteOrderQtyMarketAllowed</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>isSpotTradingAllowed</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>isMarginTradingAllowed</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>permissions</td>
      <td>VARCHAR(100)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(12)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>minPrice</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>maxPrice</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>tickSize</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(13)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>multiplierUp</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>multiplierDown</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>avgPriceMins</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(12)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>minQty</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>maxQty</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>stepSize</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(8)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>minNotional</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>applyToMarket</td>
      <td>BOOL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>avgPriceMins</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(13)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>limit_</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(15)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>minQty</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>maxQty</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>stepSize</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(14)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>maxNumOrders</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cid</th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>filterType</td>
      <td>VARCHAR(19)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>maxNumAlgoOrders</td>
      <td>INT</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>symbol_key</td>
      <td>VARCHAR(50)</td>
      <td>1</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>

