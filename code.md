#PTT Crawler
---
## 簡介  
**PTT Crawler** 是一個用於爬取 **PTT**（**批踢踢實業坊**）論壇文章的 **Python** 工具。這個程式可以幫助你從指定的 PTT 看板中爬取文章標題、連結等資訊，並將這些資料存儲到 **MongoDB 資料庫**中，方便後續分析和查閱。  

## 主要功能  
* **爬取多個看板：** 可以同時指定多個 PTT 看板，將每個看板的文章資訊爬取並存儲到對應的 MongoDB 集合中。

* **設定爬取頁數：** 為了方便測試，程式可以設定爬取每個看板的前 N 頁文章，以控制爬取的數據量。

* **資料存儲到 MongoDB：** 爬取到的文章標題、連結等資訊會被結構化地存儲到 MongoDB 資料庫中，方便後續查詢和分析。  

## 如何使用  
**1.安裝相依套件：** 在執行程式之前，需先安裝必要的套件，可透過以下指令安裝：
```htmlembedded=
pip install requests lxml pymongo
```
**2.啟動 MongoDB 伺服器：** PTT Crawler 將資料存儲到 MongoDB，因此需要確保 MongoDB 伺服器在運行中。可以使用以下指令啟動伺服器：  
```htmlembedded=
mongod --bind_ip 127.0.0.1 --port 12346
```
**3.編輯看板列表：** 在程式碼中指定想要爬取的 PTT 看板，以及設定其他相關參數。

**4.執行程式：** 執行程式以開始爬取 PTT 文章資訊。
```htmlembedded=
python ptt_crawler.py
```
## 完整程式碼
```htmlembedded=
import requests
from lxml import html
from pymongo import MongoClient
import pymongo

def crawl_ptt_collections(board_names):
    client = pymongo.MongoClient("mongodb://must:must@127.0.0.1:12346/")
    db = client['ptt']

    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36'}

    with requests.Session() as session:
        session.cookies['over18'] = '1'

        for board_name in board_names:
            i = 1

            url = f'https://www.ptt.cc/bbs/{board_name}/index1.html'

            collection = db[board_name]

            collection.delete_many({})

            while true:
                response = session.get(url, headers=headers)
                html_content = response.text
                parsed_html = html.fromstring(html_content)

                titles = parsed_html.xpath('//div[@class="title"]/a/text()')
                links = parsed_html.xpath('//div[@class="title"]/a/@href')

                for title, link in zip(titles, links):
                    article_data = {
                        '標題': title,
                        '連結': f'https://www.ptt.cc{link}',
                        '頁碼': i
                    }
                    collection.insert_one(article_data)

                next_page = parsed_html.xpath('//a[contains(text(), "下頁")]/@href')
                if next_page:
                    url = 'https://www.ptt.cc' + next_page[0]
                    i += 1 
                else:
                    print(f"已經到達 {board_name} 最後一頁")
                    break
#ptt所有的板名
board_names = ['Gossiping', 'C_Chat','Stock','Baseball','HatePolitics',
               'NBA','Lifeismoney','LOL','car','home-sale','KoreaStar',
               'basketballTW','Beauty','Tech_Job','movie','sex','MobileComm',
               'PC_Shopping','BaseballXXXX','Military','Japan_Travel','Elephants',
               'WomenTalk','creditcard','Boy-Girl','BabyMother','Steam','Tainan',
               'AllTogether','miHoYo','Kaohsiung','marriage','TaichungBun','Marginalman',
               'KoreaDrama','iOS','PuzzleDragon','japanavgirls','PlayStation','joke',
               'HardwareSale','Japandrama','CFantasy','China-Drama','e-shopping','KR_Entertain',
               'watch','Badminton','Aviation','AC_In','CVS','ONE_PIECE','Gamesale',
               'MacShop','fastfood','CarShop','Hsinchu','Soft_Job','Drama-Ticket','MLB',
               'SportLottery','biker','mobilesales','BeautySalon','Headphone','marvel',
               'Salary','ToS','Teacher','Bank_Service','WOW','Lakers','EAseries','BlueArchive',
               'Road_Running','e-coupon','MuscleBeach','PokemonGO','TY_Research','NSwitch',
               'E-appliance','StupidClown','DC_SALE','SuperBike','MH','DigiCurrency','DigiCurrency',
               'Railway','FATE_GO','Wanted','KoreanPop','Guardians','Monkeys','Food','Hearthstone',
               'YuanChuang','PublicServan','give','part-time','DIABLO','Tennis','SakaTalk','DSLR',
               'Audiophile','DMM_GAMES','GBF','MakeUp','nb-shopping','PathofExile','PathofExile',
               'TW_Entertain','Chiayi','cat','Brand','Lions','PokeMon','Gov_owned','BabyProducts',
               'HelpBuy','MRT','Coffee','FAPL','Hip-Hop','MayDay','Key_Mou_Pad','Taoyuan','facelift',
               'Digitalhome','Palmar_Drama','EatToDie']

crawl_ptt_collections(board_names)
```
