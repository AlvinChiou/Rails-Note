# Rails with Time zone
做一個國際型的社群網站多少都會碰到時區問題，而在 Rails 裡面這部分算是還蠻容易就可以處理的了。

## 取得日期與時間

在 Rails 中，通常會透過以下幾種敘述得到日期或者日期時間

```
Time.now
Date.today
```

但是這種方式取得的時間都是以「執行端」為主的時間，抓的基本上就是系統時間，所以一般在台灣的主機跑的都是 +08:00 ，假設有日本、韓國或者夏威夷等地方的使用者想要取得他們當地的時間的話用這種方式絕對會出問題。

所以就必須用另外一種方式來取得時間，就是：
```
Time.current
Date.current
```

如此一來抓到的時間點才會是根據不同時區而做過調整的。然而，有一些 API 本身就會跟著時區設定跑，像是：

```
1.days.ago
10.days.from_now
```
諸如此類這種的就會根據 Rails 的 time zone 設定而決定當前的時區，設定的方式就是在 config/application.rb 中編輯 

```
config.time_zone = 'Taipei' 
```

就可以設定台灣時區。

Rails 支援很多的時區設定，但是都是根據名稱為主，而想要有詳細的列表可以在命令列(終端機)執行 rake time:zones:all 檢查。

```
rake time:zones:all
```

然而有的時候可能會需要過濾一下一些時區，或者自定時差來列表，這時候可以用類似以下方式取得：

```
rake time:zones:us # 取得美國地區所有的可設定值 (美國跨好幾個時區)
rake time:zones:local # 取得你當前時區的所有可設定值
rake time:zones:all OFFSET=-3 # 取得時區偏移量為 -3 小時的
```

## 在 Controller 該怎麼設定

所以，當我們希望加上自訂時區設定的時候，會希望可以自動生效時區設定，這時候我們就可以用 around_action (Rails 4 之前的版本是 around_filter) 來做設定。

```
class ApplicationController < ActionController::Base
  around_filter :setup_timezone, :if => :current_user?
  
  def setup_timezone(&block)
    Time.use_zone(current_user.time_zone, &block)
  end
end
```

所以，要記得替你的 User model 加上 time_zone 這個欄位。