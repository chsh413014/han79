## 個人小研究
[Python](Research.md#拓元售票)
[LINE Notify](Research.md#line-notify)

### Python
#### 拓元售票
```
#!/usr/bin/env python
# coding: utf-8

# In[18]:


# =========預設值設定=========
def DefValue():
    # 演唱會Url  21_WuBaiKH
    #21_Eclipse
    ticketUrl = 'https://tixcraft.com/activity/detail/22_5566'
    
    # 選擇日期
    user_date = '2022/06/11'
    
    user_tktinfo = []
    # 選擇樓層 # 選擇票價  
    user_floor_1 = '綠407'
    user_price_1 = '2156'   
    
    user_floor_2 = '綠408'
    user_price_2 = '2156'
     
    user_floor_3 = '綠409'
    user_price_3 = '2156'   
    
    user_floor_3 = '綠410'
    user_price_3 = '2156'     

    # 訂票張數
    user_number = '2'
    
    user_tktinfo.append(user_floor_1 + ',' + user_price_1 + ',' + user_number) 
    user_tktinfo.append(user_floor_2 + ',' + user_price_2 + ',' + user_number) 
    user_tktinfo.append(user_floor_3 + ',' + user_price_3 + ',' + user_number)
    print(user_tktinfo)
       
    # 決定是否選擇熱賣區
    statusMark = True
  
    query_headers = {
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'Accept-Encoding': 'gzip, deflate, br',
        'Accept-Language': 'zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-CN;q=0.6',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
        'Pragma': 'no-cache',
        'sec-ch-ua': '" Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"',
        'sec-ch-ua-mobile': '?0',
        'Sec-Fetch-Dest': 'document',
        'Sec-Fetch-Mode': 'navigate',
        'Sec-Fetch-Site': 'none',
        'Sec-Fetch-User': '?1',
        'Upgrade-Insecure-Requests': '1',
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36'

    }
    
    cookies = {"SID": "1854pm13385ce8crijluh55s34", }

    print("========== 預設值設定 ==========")
    print("網址：" + ticketUrl)
    print("日期：" + user_date)
    print("樓層：" + str(user_tktinfo))
    #print("票價：" + user_price)
    print("張數：" + user_number)
    
    return ticketUrl, query_headers, cookies, user_date, user_tktinfo, user_number

#DefValue()


# In[2]:


# ========= 查詢 =========
def QueryTicket(ticketUrl, headers, cookies):
       
    ticketUrl = ticketUrl.replace('detail','game')
    
    #ticket
    print("========== Session ==========")
    #將request 存入session
    sq = requests.Session()  
    
    resp  = sq.get(ticketUrl, headers=headers, cookies=cookies)
    if resp.status_code != 200:
        print('URL發生錯誤：' + url)
    print('url : ' + str(resp.url))
    
    soup  = BeautifulSoup(resp.text, 'html5lib') 
    
    
    return soup, sq


# In[3]:


# ========= 登入會員 =========
def checkLogin(soup):
        
    print("========== 確認是否登入 ==========")
    #印出確認是否登入
    try:
        username  = soup.find('a','user-name').text
        print('使用者:' + soup.find('a','user-name').text)
    except:
        username  = soup.find('li','account-login').a.text.strip()
        print(username + "/尚未登入tixCraft")
    
    return username
    #session key
    #if username != "會員登入":
        #csrf   = soup.cookies["CSRFTOKEN"]
        #lang   = soup.cookies["lang"]
        #cookie = soup.request.headers['Cookie']


# In[4]:


# ========= 檢查日期是否已有開賣按鈕 =========
def gameList(soup):
    
    print("========== 檢查日期是否已有開賣按鈕 ==========")
    check = False
    gameDic = {}
    gameList = soup.find('div',id='gameList').find_all('tr','gridc fcTxt')
    for trs in gameList:
        #print(trs.find_all('td')[3].find_all('input'))
        if trs.find_all('td')[3].find_all('input') == []:
            print(trs.find_all('td')[0].text[:10] + ' ' + trs.find_all('td')[3].text)
            #checkTxt = trs.find_all('td')[3].text
        else:
            date = trs.find_all('td')[0].text[:10]
            href = trs.find_all('td')[3].find_all('input')[0]['data-href']
            gameDic[date] = href
            #checkTxt = '' 
            check = True
            #gameDic.setdefault(Date,Name) #['date'].append(Date)
            #gameDic['name'].append(Name)
    # if user_date in gameDic.keys() == False:
    #checkDate = user_date in gameDic.keys()
    print('演唱日期列表：' + str(gameDic))
    
    return check, gameDic


# In[5]:


# 透過User選擇的日期判斷是否存在並回傳資料
def GetUrl(gameDic, user_date):
    url = ''
    if (user_date in gameDic.keys()) == True:      
        url = 'https://tixcraft.com' + gameDic[user_date]
        checkdate = True
    else:
        checkdate = False
        
    return url, checkdate


# In[6]:


# 選位資料整理
def areaTicket(areaUrl, sq, headers, cookies):
    #print(areaUrl)
    areaResp = sq.get(areaUrl, headers=headers, cookies=cookies)
    if areaResp.status_code != 200:
        print('URL發生錯誤：' + url)
   
    areaSoup = BeautifulSoup(areaResp.text, 'html5lib')
    #print(areaSoup)
    # 可選座位陣列
    areaDic  = {}
    areaList = re.findall(r"areaUrlList = {[^}]+}", areaSoup.text)[0]
    areaList = areaList.replace("\\", "").replace("areaUrlList = ", "")
    areaList = areaList.replace("{","").replace("}","").replace('"','')
    #print(areaUrlList)
    for strSeat in areaList.split(','):
        SeatId  = strSeat.split(':')[0]
        SeatUrl = strSeat.split(':')[1]
        areaDic[SeatId] = SeatUrl
    #print(areaDic) 
    
    # 可選座位的文字陣列
    areaText = []
    area_ids  = areaSoup.find('div','zone area-list').find_all('ul','area-list')
    #print(area_ids) 
    for area_li in area_ids:
        #print(area_li)
        #print('------------------------------')        
        areali_ids = area_li.find_all('li')
        for area in areali_ids:
            #print(area)
            #print('====================')
            if str(area.a) != 'None': 
                area_names   = area.a.text.strip()
                #票價的key值
                area_id      = area.a['id']
                #票價
                area_price   = int(re.findall(r"\d\d\d+", area_names)[-1])
                #剩餘票數及狀態
                areas_status = area.a.font.text.strip() 
                #座位區域
                area_name    = area_names.replace(str(area_price), "").replace(areas_status, "").strip()
                # id, 區域, 票價, 狀態
                area_info    = area_id + ',' + area_name + ',' + str(area_price) + ',' + areas_status
                areaText.append(area_info) 
    print(areaText)   
    print('====================')
    return areaSoup,areaDic,areaText
    


# In[20]:


# 選位邏輯
def areaSeat(areaDic, areaTexts, user_tktinfo):
    for tkt in user_tktinfo:
        ticketUrl = ''
        print("票價優先順序 = " + tkt)
    
        user_floor = tkt.split(',')[0]
        user_price = tkt.split(',')[1]      
        user_Num = tkt.split(',')[2] 
        
        #取得最後一筆資料
        #print(areaTexts[len(areaTexts)-1])
        last_id = areaTexts[len(areaTexts)-1].split(',')[0]
        #print(last_id)
        for areaArray in areaTexts:
            print(areaArray)
            area_id     = areaArray.split(',')[0]
            area_name   = areaArray.split(',')[1]
            area_price  = areaArray.split(',')[2]
            area_status = areaArray.split(',')[3]  
            area_tick   = int(re.findall(r"\d", areaArray.split(',')[3])[-1])
            #print('====================')
            #print(area_tick)
            #print('url = '+ticketUrl) 
            stop = 1
            statusMark = True
            while ticketUrl == '': 
                stop += 1
                # 是否優先取 熱賣中
                #if area_status.find('熱賣中') != -1 and statusMark == True:                
                # 區域順位
                if area_name.find(user_floor) != -1:
                    #票價順位
                    if str(area_price).find(user_price) != -1:
                        #print(area_id)
                        ticketUrl = areaDic[area_id]
                    
                # 取到 url就跳出，不繼續跑
                if ticketUrl != '':
                    #print('lastUrl = '+ticketUrl)
                    break        
                
                if area_id == last_id:
                    #statusMark = False
                    #print('last_id = '+area_id)
                    break
                
                if stop > 20:
                    #statusMark = False
                    break
                    
            # 取到 url就跳出，不繼續跑
            if ticketUrl != '':
                #print('last2Url = '+ tkt)
                break    
        
        # 若第一筆就娶到url，就不繼續抓下一筆資料      
        if ticketUrl != '':
            break
                #elif str(area_price).find('2000') != -1:
                #    ticketUrl = areaDic[area_id] 
    
    if ticketUrl == '':
        area_id = areaTexts[0].split(',')[0]
        ticketUrl = areaDic[area_id]
        
    return 'https://tixcraft.com' + ticketUrl         
                
#areaSeat(areaDic, areaText, user_tktinfo) 


# In[8]:



def GetTicket(ticketUrl, sq, headers, cookies):
    # ticket_ticket
    ticketResp = sq.get(ticketUrl, headers=headers, cookies=cookies)
    if ticketResp.status_code != 200:
        print('URL發生錯誤：' + url)
    #ticketRespUrl = ticketResp.url 
    
    ticketSoup = BeautifulSoup(ticketResp.text, 'html5lib')  
    return ticketSoup, ticketResp


# In[9]:


def GetData(ticketSoup, ticketResp, user_number):
    
    # 選擇的區域
    ticketarea = ticketSoup.find('div','flow-content').find('div','select-area').p.find_all('span')[0].text.strip()
    ticketnu = ticketSoup.find('div','flow-content').find('div','select-area').p.find_all('span')[2].text.strip()
    print('所選擇區域：' + ticketarea + '，' + ticketnu)
    
    # CSRFTOKEN
    CSRFTOKEN = ticketSoup.find(id ='TicketForm').find('input',{'name':'CSRFTOKEN'})['value']
    #print("訂票TOKEN：" + CSRFTOKEN)
    
    # ticketPrice 張數
    ticketPrice    = ticketSoup.find('tr','gridc').find('select','mobile-select')['name']
  
    option_numbers = ticketSoup.find('select','mobile-select').find_all('option')    
    #預設最大張數
    for option in option_numbers:   
        option_number = option['value']
    
    if user_number == '':   
        option_number = option_number
    elif option_number < user_number:
        option_number = option_number
    else:        
        option_number = user_number
    print("訂票張數：" + option_number)
       
    # priceSize
    priceSize  = ticketSoup.find(id = 'ticketPriceList').find('input',{'type':'hidden'})['name']
    priceValue = ticketSoup.find(id = 'ticketPriceList').find('input',{'type':'hidden'})['value']
    #print("價錢單位：" + priceSize + ',' + priceValue)
    
    # [agree][MQjWMFfVWby/OpDBUxww7c28WGZBINyItmpjyYkPsWo=] 同意驗證
    agree      = re.findall(r"TicketForm\[agree]\[\S+\]", ticketResp.text)[0]
    agreeValue = ticketSoup.find(id = 'TicketForm_agree')['value']
    
    return CSRFTOKEN, ticketPrice, option_number, priceSize, priceValue, agree
#GetData(ticketSoup, ticketResp, user_number)


# In[10]:


def Captcha(ticketsoup, sq, headers, cookies):
    
    #VerifyCode 圖形驗證
    imgUrl = ticketsoup.find('div','verify-img').img['src']
    #print(url + imgUrl)
    imgResp = sq.get('https://tixcraft.com' + imgUrl, headers=headers, cookies=cookies)
    img = Image.open(BytesIO(imgResp.content)) 

    img.show()
    #plt.imshow(img)
    #plt.show() 


# In[11]:


def CaptchaKeyin():
    CaptchaKey = input("請輸入驗證碼：")
    return CaptchaKey


# In[12]:


def DataSet(CSRFTOKEN, ticketPrice, option_number, priceSize, priceValue, verify, agree):
    #ticket_ticket 
    formData = {
        "CSRFTOKEN": CSRFTOKEN,
        ticketPrice: option_number,
        priceSize: priceValue,
        "TicketForm[verifyCode]": verify,
        agree: "1",
        "ticketPriceSubmit": "", 
    } 
    #print(formData)
    
    return formData

#formData = formData(CSRF, ticketPrice, option_number, priceSize, priceValue, CaptchaKey, agree)


# In[13]:


def DataPost(ticketUrl, formData, sq, headers, cookies):
    
    postResp = sq.post(ticketUrl, headers=headers, cookies=cookies, data=formData)
    postSoup = BeautifulSoup(postResp.text, 'html5lib') 
    print('  status : ' + str(postResp.status_code))
    print('Post_url : ' + str(postResp.url))
    #print(postsoup)
    alertText = re.findall(r"alert\(\S+\)", postResp.text)
    msg = ''
    if len(alertText) > 1:
        msg = alertText[len(alertText)-1] #取最後一個資料
        msg = msg.replace("alert(\"", "").replace("\")","")  
        msg = msg.encode('latin-1').decode('unicode_escape')
        print(msg)

    if msg == "您所輸入的驗證碼不正確，請重新輸入":
        error = 'err'
    else:
        error = 'yes'
        
    return postSoup, postResp, postResp.url, error


# In[14]:


def OrderQuery(url, sq, headers, cookies):  
    
    orderResp = sq.get(url, headers=headers, cookies=cookies )
    print('order_url : ' + orderResp.url)

    checkResp = sq.get('https://tixcraft.com/ticket/check', headers=headers, cookies=cookies )
    checksoup = BeautifulSoup(checkResp.text, 'html5lib') 
    print('check_url : ' + checkResp.url)
    #print(checksoup)  
    
    json_data = json.loads(checkResp.text)
    message   = json_data["message"]
    time      = int(json_data["time"])
    print(json_data)
    #print("=========")

    if message.find('alert(') != -1:
        msg = re.findall(r"alert\(\"[^\"]+", message)[0]
        msg = msg.replace("alert(\"", "")    
        msg = msg.encode('latin-1').decode('unicode_escape')
    else:
        msg = message
        
    print(msg)
    print("===========================")
    if time == 0:
        print(msg)
        print(f"請等待: {time}秒")
    else:
        print(msg)
        print(f"請等待: {time}秒")
        sleep(time)  


# In[15]:


def Payment(sq, headers, cookies):
    # 若遇到等五秒 要讓他睡一下
    paymentResp = sq.get('https://tixcraft.com/ticket/payment', headers=headers, cookies=cookies )
    paymentsoup = BeautifulSoup(paymentResp.text, 'html5lib') 
    print('payment_url : ' + paymentResp.url)  


# In[16]:


def OrderList(sq, headers, cookies):
    sleep(3)
    checkorderResp = sq.get('https://tixcraft.com/ticket/order', headers=headers, cookies=cookies )
    checkordersoup = BeautifulSoup(checkorderResp.text, 'html5lib') 
    print('last_url : ' + checkorderResp.url)
    #print(checkordersoup)
    checkCentent = checkordersoup.find('section','mainContent')
    #print(checkCentent)
    #tb_row order_list 

    #目前無訂單資訊
    for NoTicket in checkCentent.find_all('tbody'):
        print(NoTicket.tr.td.text)

    for OrderRow in checkCentent.find_all('div','tb_row order_list'):
        # 訂單編號
        Order_No = OrderRow.find('div','order_no')
        print("訂單編號:" + Order_No.span.text.strip())

        # 購買日期
        Order_Date = OrderRow.find('div',{"data-title":"訂購日期"})
        print("購買日期:" + Order_Date.text.strip())

        # 購買節目
        Event_Info = OrderRow.find('div','event_info_name')
        print("購買節目:" + Event_Info.text.strip())

        # 訂單狀態
        Go_Pay = OrderRow.find('div','go_pay')
        print("購買節目:" + Go_Pay.text.strip())
    print ("------------------------")
        #票價資訊
    for OrderLists in checkCentent.find_all('div','tb_row repo_bg'):   
        for OrderList in OrderLists.find_all('div'):
            print (OrderList.text.strip())
        print ("------------------------")

# In[ ]:

# In[ ]:
def wait_start():
    print("  now : " + time.ctime())
    print("start : " + datetime.datetime(START_DATE[0], START_DATE[1], START_DATE[2], 
                                         START_DATE[3], START_DATE[4],START_DATE[5]).ctime())
    
    now = time.localtime()
    
    difference = datetime.datetime(START_DATE[0], START_DATE[1], START_DATE[2], START_DATE[3], START_DATE[4],START_DATE[5]) - datetime.datetime(now[0], now[1], now[2], now[3], now[4], now[5])
    
    print("difference.seconds : " + str(difference.seconds))
     
    time.sleep(difference.seconds - 1)
        
    print(time.ctime())
# In[ ]:


import re
import requests
import json
from time import sleep
from bs4 import BeautifulSoup #爬蟲模組
from requests.cookies import RequestsCookieJar 

import matplotlib.pyplot as plt
import matplotlib.image as img
from PIL import Image 
from io import BytesIO
import sys

import time
import datetime
#  獲得當前時間
#now = datetime.datetime.now() #2019-04-11 14:18:41.629019
#print(now)

#TimeNow()
# 等待開始
WAIT_START = True
START_DATE = [2022, 4, 17, 12, 0, 0]  # YYYY,MM,dd,HH,mm

print(START_DATE)

if WAIT_START:
    wait_start()
    
print("Start")

#sys.exit(0)


# 初始設定
Url, headers, cookies, user_date, user_tktinfo, user_number = DefValue()
# 查詢演唱會
soup, sq = QueryTicket(Url, headers, cookies)
# 檢查登入
username = checkLogin(soup)

print("  now : " + time.ctime())
if username == "會員登入":
    sys.exit(0)
    
if username != "會員登入":
    
    # 檢查日期是否已有開賣按鈕
    check, gameDic = gameList(soup)
    
    # 如果不存在為 尚未開賣/已售完 若已售完折停止
    stop = 1
    while check == False :
        stop += 1

        soup, sq = QueryTicket(Url, headers, cookies)
        check, gameDic = gameList(soup)

        print('while=' + str(stop) + ',check=' + str(check))
        
        if stop > 10:
            print('if='+str(stop) + ',check=' + str(check))
            print('====== 預設10次停止查詢 ======')
            break    
            
    # 已確認存在後·抓取使用者輸入的座位網址
    if check == True:
        areaUrl, checkdate = GetUrl(gameDic, user_date)
        
        stop = 1
        while checkdate == False:            
            print('===== 查無演唱日期！請重新填寫！')
            
            user_date = input('演唱日期：')
            areaUrl, checkdate = GetUrl(gameDic, user_date)
            
        print(areaUrl)
         
    # 進入選位
    if areaUrl != '':
        #print(user_date)
        # 選位資料整理
        areaSoup, areaDic, areaText = areaTicket(areaUrl, sq, headers, cookies)
        # 選位機制
        ticketUrl = areaSeat(areaDic, areaText, user_tktinfo) 
        print(ticketUrl)     
        # GetQuery
        ticketSoup, ticketResp = GetTicket(ticketUrl, sq, headers, cookies)
        
        checkErr = ''
        while checkErr == '' or checkErr == 'err':
            # 解析資料包
            CSRF, ticketPrice, option_number, priceSize, priceValue, agree = GetData(ticketSoup, ticketResp, user_number)
            # 顯示驗證碼
            Captcha(ticketSoup, sq, headers, cookies)       
            # 驗證碼keyin
            CaptchaKey = CaptchaKeyin()
            # 資料包
            formData = DataSet(CSRF, ticketPrice, option_number, priceSize, priceValue, CaptchaKey, agree)
            # 發送資料包
            ticketSoup, ticketResp, orderUrl, checkErr = DataPost(ticketUrl, formData, sq, headers, cookies)
        
        #print(orderUrl)
        if checkErr == 'yes':
            #
            OrderQuery(orderUrl, sq, headers, cookies)
            # 進入付款頁面
            Payment(sq, headers, cookies)
            # 顯示訂單資料
            OrderList(sq, headers, cookies)
        #print(ticketRespUrl)

```

### LINE Notify
登入到 LINE Notify 並進入到個人頁面

接著選擇要接收通知的聊天室，也可以透過一對一接收 LINE Notify 的通知

把產生的權證記錄下來，如果遺失或忘記的話就只能重新連接獲取新的權證

連結Google雲端行事曆
可依照行事曆顏色型別做不同通知設定

```
// 實際發送訊息
function SendTkt(message, token) {
 var options =
   {
     "method"  : "post",
     "payload" : {"message" : message},
     "headers" : {"Authorization" : "Bearer " + token}
   };
UrlFetchApp.fetch("https://notify-api.line.me/api/notify", options); 
}
```
#### 會議通知
```
// 請記得填寫週二EBS更新紀錄 (星期一下午發送通知)
function EBSMsg(token)
{
  var Calendar_ID = 'b5e9c8a94718850b8b9c5573f5769c900eda5e6b6a01@group.calendar.google.com'
  var token       = "b3dOKslhk2DIRWIWb4SXH9l6uSaoGtzn2";  //票務小組
  var message     = "";

  let cal = CalendarApp.getCalendarById(Calendar_ID);

  var date = new Date();
  var secondDate = new Date(date.getFullYear(), date.getMonth(), date.getDate()+1);
  var firstDay = new Date(date.getFullYear(), date.getMonth(), 1);
  var lastDay = new Date(date.getFullYear(), date.getMonth() + 1, 0);
  var events = cal.getEvents(firstDay, lastDay);
  for(let event of events)
  {
    Logger.log(secondDate.toDateString());
    Logger.log(event.getStartTime().toDateString());
    Logger.log(event.getStartTime());

    if(secondDate.toDateString() == event.getStartTime().toDateString())
    {
      Logger.log(event.getColor());
      Logger.log(event.getDescription());
      if(event.getColor() == 4 ) // 粉紅 - 請記得填寫週二EBS更新紀錄
      {
        var href = event.getDescription().split('"');
        Logger.log(href[1]);
     
        message = event.getTitle() + "\r\n" + href[1];
        SendTkt(message, token);
      }
      break;
      //Logger.log(event.getTitle());
    }
  }
}

// 票務小組-進度及會議通知
function DayTktMsg(token)
{
  var Calendar_ID = 'c8a94718850b8b9c5573f5769c900eda5e6b6a01@group.calendar.google.com'
  var token       = "DIRWIWb4SXH9l6uSaoGtzn2R5fKCb2SGH"; //票務小組
  var message     = "";

  let cal = CalendarApp.getCalendarById(Calendar_ID);

  var date = new Date();
  var firstDay = new Date(date.getFullYear(), date.getMonth(), 1);
  var lastDay = new Date(date.getFullYear(), date.getMonth() + 1, 0);
  var events = cal.getEvents(firstDay, lastDay);
  for(let event of events)
  {
   // Logger.log(date.toDateString());
   // Logger.log(event.getStartTime().toDateString());
   // Logger.log(event.getStartTime());

    if(date.toDateString() == event.getStartTime().toDateString())
    {
      //Logger.log(event.getColor());
      if(event.getColor() == 5 ) // 黃色 - 請給每兩週工作進度/會議/教育訓練
      {
        message = event.getTitle();
        SendTkt(message, token);
      }
      if(event.getColor() == 11) // 紅色 - 繳交每月5號進度報告
      {
        Logger.log(event.getTitle());
        message = event.getTitle();
        SendTkt(message, token);
      }   
      break;
      //Logger.log(event.getTitle());
    }
  }
}

```
#### 連接google excel表單
```
// 設定事件一
function myMessage()
{
  var myToken     = 'WMD8avp1fMd5XYNdJ7UwhVnE'
  var mySheetUrl  = 'https://docs.google.com/spreadsheets/d/1GWlTLy30upE1LBlsMl9A62sl0BajkvskjbQsLAj4y30/edit#gid=0'
  var mySheetName = '工作表1'
  var SpreadSheet = SpreadsheetApp.openByUrl(mySheetUrl);
  var SheetName   = SpreadSheet.getSheetByName(mySheetName);
  var myMsg       = SheetName.getSheetValues(2,2,1,1)[0].toString();
  //Logger.log(SheetName.getSheetValues(2,2,1,1)[0]);
  doPost(myMsg, myToken);
}


// 店長鴨血通知
function doPost(msg, token) {
  UrlFetchApp.fetch('https://notify-api.line.me/api/notify', {
        'headers': {
           'Authorization': 'Bearer ' + token,
        },
        'method': 'post',
        'payload': {
            'message':msg
        }
    });
}
```
