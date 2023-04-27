---
title: "Stock Secretary"
author: "Jang Su Kyung"
date: "2023-03-24"
---
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
<br />

#### 주식 정보에 대해 크롤링해서 알려주는 비서를 만들었습니다.
#### 주식 정보 검색기부터 데이터를 바탕으로 일봉 차트를 만드는 것까지 진행했습니다.
<br />

# **1. 주식 정보 검색기**
```{python}
import requests
from bs4 import BeautifulSoup
import json

name = input()

def get_stock_code(name):
    url = f"https://ac.finance.naver.com/ac?_callback=window.__jindo2_callback._$3361_0&q={name}&q_enc=euc-kr&st=111&frm=stock&r_format=json&r_enc=euc-kr&r_unicode=0&t_koreng=1&r_lt=111"
    response = requests.get(url).text
    soup = json.loads(response[response.find('{'):response.rfind('}')+1])
    items = soup['items']
    code = items[0][0][0][0]
    return code
s_code = get_stock_code(name)
```
<br />

#### 1) 네이버 증권에서 종목명을 바탕으로 종목 코드를 가져오는 명령어입니다.
#### 2) json을 이용해 업로드되는 javascript 파일을 추출해서 코드를 가져왔습니다.
<br />

# **2. 종목 요약 정보**
```{r}
def stock_summary(name):
    url=f"https://finance.naver.com/item/main.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)
    soup=BeautifulSoup(res.text,"lxml")
    
    print(f"########## {name} 요약 정보 ##########")
    print()
    # 시가총액
    stock_mc = soup.select_one('#_market_sum').text.strip()
    stock_mc = stock_mc.replace('\t', '').replace('\n', '').replace(',', '')
    print(f"\033[31m\033[91m시가총액\033[0m은 \033[31m\033[93m{stock_mc}억 원\033[0m입니다.")

    # 시가총액 순위
    stock_mc_rank = soup.select_one('.first tr:nth-child(3) td').text
    print(f"\033[31m\033[91m시가총액 순위\033[0m는 \033[31m\033[93m{stock_mc_rank}\033[0m입니다.")

    # 상장주식수
    stock_qs = soup.select_one('.first tr:nth-child(4) em').text
    print(f"\033[31m\033[91m상장주식수\033[0m는 \033[31m\033[93m{stock_qs}\033[0m입니다.")

    # 액면가
    stock_face_value = soup.select_one('#tab_con1 tr:nth-child(5) em:nth-child(1)').text
    print(f"\033[31m\033[91m액면가\033[0m는 \033[31m\033[93m{stock_face_value}원\033[0m입니다.")

    # 외국인한도주식수(A)
    stock_foreign_A = soup.select_one('.lwidth tr:nth-child(2) em').text
    print(f"\033[31m\033[91m외국인한도주식수(A)\033[0m는 \033[31m\033[93m{stock_foreign_A}주\033[0m 입니다.")

    # 외국인한도주식수(B)
    stock_foreign_B = soup.select_one('.lwidth tr:nth-child(3) em').text
    print(f"\033[31m\033[91m외국인한도주식수(B)\033[0m는 \033[31m\033[93m{stock_foreign_B}주\033[0m 입니다.")

    # 외국인소진율(B/A)
    stock_foreign_er = soup.select_one('.lwidth tr:nth-child(4) em').text
    print(f"\033[31m\033[91m외국인소진율(B/A)\033[0m는 \033[31m\033[93m{stock_foreign_er}\033[0m 입니다.")

    # 투자의견
    try:
        stock_io = soup.select_one('#tab_con1 .f_up').text
        print(f"\033[31m\033[91m투자의견\033[0m은 \033[31m\033[93m{stock_io} 의견\033[0m입니다.")
    except:
        print("\033[31m\033[91m투자의견\033[0m이 없습니다.")

    # 목표주가
    try:
        stock_tp = soup.select_one('.rwidth tr:nth-child(2) em:nth-child(3)').text
        print(f"\033[31m\033[91m목표주가\033[0m는 \033[31m\033[93m{stock_tp}원\033[0m 입니다.")
    except:
        print("\033[31m\033[91m목표주가\033[0m가 없습니다.")

    # 52주 최고가
    stock_52_high = soup.select_one('.rwidth tr:nth-child(3) em').text
    print(f"\033[31m\033[91m52주 최고가\033[0m는 \033[31m\033[93m{stock_52_high}원\033[0m 입니다.")

    # 52주 최저가
    stock_52_row = soup.select_one('.rwidth tr:nth-child(3) em:nth-child(3)').text
    print(f"\033[31m\033[91m52주 최저가\033[0m는 \033[31m\033[93m{stock_52_row}원\033[0m 입니다.")

    # PER / EPS
    stock_per = soup.select_one('#_per').text
    stock_eps = soup.select_one('#_eps').text
    print(f"\033[31m\033[91mPER\033[0m은 \033[31m\033[93m{stock_per}배이고 \033[31m\033[91mEPS\033[0m는 \033[31m\033[93m{stock_eps}원\033[0m 입니다.")

    # 추정 PER / 추정 EPS
    try:
        stock_cns_per = soup.select_one('#_cns_per').text
        stock_cns_eps = soup.select_one('#_cns_eps').text
        print(f"\033[31m\033[91m추정 PER\033[0m은 \033[31m\033[93m{stock_cns_per}배\033[0m이고 \033[31m\033[91m추정 EPS\033[0m는 \033[31m\033[93m{stock_cns_eps}원\033[0m 입니다.")
    except:
        print("\033[31m\033[91m추정치가 없습니다.")
    # PBR / BPS
    stock_pbr = soup.select_one('#_pbr').text
    stock_bps = soup.select_one('.per_table tr:nth-child(4) em:nth-child(3)').text
    print(f"\033[31m\033[91mPBR\033[0m은 \033[31m\033[93m{stock_pbr}배이고 \033[31m\033[91mBPS\033[0m는 \033[31m\033[93m{stock_bps}원\033[0m 입니다.")

    # 배당수익률
    try:
        stock_dir = soup.select_one('#_dvr').text
        print(f"\033[31m\033[91m배당수익률\033[0m은 \033[31m\033[93m{stock_dir}%\033[0m 입니다.")
    except:
        print("\033[31m\033[91m배당수익률\033[0m이 없습니다.")
    # 동일업종 PER
    stock_si_per = soup.select_one('.gray .strong:nth-child(2) em').text
    print(f"\033[31m\033[91m동일업종 PER\033[0m은 \033[31m\033[93m{stock_si_per}배\033[0m 입니다.")

    # 동일업종 등락률
    try:
        stock_sifr = soup.select_one('.gray .f_up em').text.strip()
        print(f"\033[31m\033[91m동일업종 등락률\033[0m은 \033[31m\033[93m{stock_sifr}\033[0m 입니다.")
    except:
        print("\033[31m\033[91m동일업종 등락률\033[0m이 없습니다.")

    print()
    print("#"*42)

stock_summary(name)
```
<br />

#### 1) 네이버 증권에서 종목명을 입력하고 들어갈 시 나오는 종목 요약 정보를 크롤링하였습니다.
#### 2) 주요 정보를 조금 더 잘 보이게 하기 위해서 빨간색과 노란색을 넣어 출력했습니다.
<br />

# **3. 종목 최신 뉴스**
```{r}
def stock_news(name):
    url=f"https://finance.naver.com/item/main.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)
    soup=BeautifulSoup(res.text,"lxml")

    print()
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m최신 뉴스\033[0m에 대해 알려드리겠습니다. \n")
    news_list = []
    index = 1
    for i in range(1,10):
        try:
            news_list=soup.select_one(f".news_section ul:nth-child(2) li:nth-child({i}) a:nth-child(1)")
            print(index,".", news_list.get_text())
            print("https://finance.naver.com"+ news_list.get("href"))
            print()
            index += 1
        except:
            pass

stock_news(name)
```
<br />

#### 1) 네이버 증권 종목 검색 시 볼 수 있는 종목 관련 최신 뉴스를 크롤링했습니다.
<br />

# **4. 종목 토론방**
```{r}
def stock_event_discussion(name):
    url=f"https://finance.naver.com/item/main.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)
    soup=BeautifulSoup(res.text,"lxml")

    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m종목 토론방 게시글\033[0m입니다.")
    print()
    news_list = []
    index = 1
    for i in range(1,10):
        try:
            news_list=soup.select_one(f".right ul:nth-child(2) li:nth-child({i}) a")
            print(index,".", news_list.get_text().strip())
            print("https://finance.naver.com"+ news_list.get("href"))
            index += 1
            print()
        except:
            pass

stock_event_discussion(name)
```
<br />

#### 1) 종목 토론방에 올라오는 게시물도 크롤링하였습니다.
<br />

# **5. 연간 재무제표**
```{r}
# 기업 연간 재무제표 출력
import pandas as pd
import requests
def stock_annual_fi(name):
    url=f"https://finance.naver.com/item/main.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)
    df = pd.read_html(res.text)[3]

    df.set_index(df.columns[0], inplace = True)
    df.index.rename('주요재무정보', inplace = True)
    df.columns = df.columns.droplevel(2)
    annual_date = pd.DataFrame(df).xs('최근 연간 실적', axis = 1)
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m최근 연간 실적\033[0m에 대해 알려드리겠습니다.")
    display(annual_date)

stock_annual_fi(name)
```
<br />

#### 1) pandas를 사용해 연간 재무제표를 크롤링하였습니다.
<br />

# **6. 분기 재무제표**
```{r}
import pandas as pd
import requests
def stock_quarter_fi(name):
    url=f"https://finance.naver.com/item/main.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)
    df = pd.read_html(res.text)[3]

    df.set_index(df.columns[0], inplace = True)
    df.index.rename('주요재무정보', inplace = True)
    df.columns = df.columns.droplevel(2)
    quarter_date = pd.DataFrame(df).xs('최근 분기 실적', axis = 1)
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m최근 분기 실적\033[0m에 대해 알려드리겠습니다.")
    display(quarter_date)

stock_quarter_fi(name)
```
<br />

#### 1) pandas를 사용해 분기 재무제표를 크롤링하였습니다.
<br />

# **7. 주요 시세 **
```{r}
# 오늘 주요 시세
import requests
import pandas as pd

def stock_major_price(name):
    url=f"https://finance.naver.com/item/sise.naver?code={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res=requests.get(url,headers=headers)

    df = pd.read_html(res.text, encoding = 'euc-kr')[1]
    df = df.dropna()
    
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m주요 시세\033[0m에 대해 알려드리겠습니다.")
    display(df)

stock_major_price(name)
```
<br />

#### 1) 네이버 증권에 있는 종목 주요 시세를 가져왔습니다.
<br />

# **8. 전자 공시 **
```{r}
import requests
import pandas as pd
def stock_public_an(name):
    url=f"https://dart.fss.or.kr/dsab001/search.ax?textCrpNM={s_code}"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"}
    res = requests.get(url, headers = headers)
    df = pd.read_html(res.text, encoding = 'euc-kr')[0]
    df = df.style.hide(axis = 'index')
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m공시 내용\033[0m에 대해 알려드리겠습니다.")
    display(df)

stock_public_an(name)
```
<br />

#### 1) pandas를 사용해 종목의 전자 공시 내용에 대해 크롤링하였습니다.
<br />

# **9. 일별 시세 **
```{r}
import pandas as pd
import matplotlib.pyplot as plt
import requests
from datetime import datetime
from matplotlib import dates as mdates
from bs4 import BeautifulSoup as bs
def stock_market_price(name):
    df = pd.DataFrame()
    for page in range(1, 20):
        url = f"https://finance.naver.com/item/sise_day.nhn?code={s_code}&page={page}"
        headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36'}
        response = requests.get(url, headers=headers)
        html = bs(response.text, 'html.parser')
        html_table = html.select("table")
        table = pd.read_html(str(html_table))
        df = pd.concat([df,table[0].dropna()])
        
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m일별 시세\033[0m에 대해 알려드리겠습니다.")
    display(df)
stock_market_price(name)
```
<br />

#### 1) 일별 시세에 대해 크롤링 하였습니다.
<br />

# **10. 일봉 차트 **
```{r}
import pandas as pd
import matplotlib.pyplot as plt
import requests
from datetime import datetime
from matplotlib import dates as mdates
from bs4 import BeautifulSoup as bs

def stock_chart(name):
    df = pd.DataFrame()
    for page in range(1, 20):
        url = f"https://finance.naver.com/item/sise_day.nhn?code={s_code}&page={page}"
        headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36'}
        response = requests.get(url, headers=headers)
        html = bs(response.text, 'html.parser')
        html_table = html.select("table")
        table = pd.read_html(str(html_table))
        df = pd.concat([df,table[0].dropna()])
        df = df.iloc[0:30]
        df = df.sort_values(by='날짜')

    plt.figure(figsize=(15, 6))
    plt.title('Chart')
    plt.xticks(rotation=45)
    plt.plot(df['날짜'], df['종가'], color = 'red', linestyle = '-', marker = 'D')
    plt.grid(color = 'gray', linestyle='--')
    print(f"\033[31m\033[91m{name}\033[0m의 \033[31m\033[93m일일 차트\033[0m입니다.")
    display(plt.show)
stock_chart(name)
```
<br />

#### 1) 9번의 일별 시세를 바탕으로 직접 일봉 차트를 만들어 보았습니다.
<br />

# **11. 주식 정보 비서 **
```{r}
def stock_searching():
    try:
        print("검색하고 싶은 종목명을 검색하세요." + "\n")
        name = input()
        global s_code
        s_code = get_stock_code(name)
        print(f"\033[31m\033[91m{name}\033[0m의 종목코드는 \033[31m \033[93m{s_code}\033[0m입니다. \n")

        while True:
            print("알고 싶은 정보를 검색하세요." + "\n")
            print("#" * 85 + "\n")
            print("요약 | 뉴스 | 토론 | 연간 실적 | 분기 실적 | 주요 시세 | 공시 | 일별 시세 | 차트 | 종료" + "\n")
            print("#" * 85 + "\n")
            stock_search = input()
            if stock_search == "요약":
                stock_summary(name)
            elif stock_search == "뉴스":
                stock_news(name)
            elif stock_search == "토론":
                stock_event_discussion(name)
            elif stock_search == "연간 실적":
                stock_annual_fi(name)
            elif stock_search == "분기 실적":
                stock_quarter_fi(name)
            elif stock_search == "주요 시세":
                stock_major_price(name)
            elif stock_search == "공시":
                stock_public_an(name)
            elif stock_search == "일별 시세":
                stock_market_price(name)
            elif stock_search == "차트":
                stock_chart(name)
            elif stock_search == "종료":
                print("종료하겠습니다.")
                break
    except:
        print("없는 종목입니다. 다시 실행시켜주세요.")

stock_searching()
```
<br />

#### 1) 1번부터 10번까지의 내용을 바탕으로 주식 정보 개인 비서를 만들었습니다.
#### 2) 종료를 입력하지 않으면 계속해서 정보를 물어볼 수 있게 했습니다.
#### 3) 이것으로 주식 종목명만 입력하면 종목에 대한 각종 정보를 볼 수 있게 되었습니다.
