## 個人小研究

### Python
#### 高鐵售票

#### 台鐵售票

#### 拓元售票


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
