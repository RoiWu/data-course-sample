# Collaborative filtering.ipynb

=====================
分數疊代過程
(rule base) => 0.132
(content base with TXT) => 0.132
(content base with Img) => 0.169
(collaborative filter) => 0.169
=====================

使用邏輯如下

- 資料清理
1. 把matedata重複asin的資料刪除
2. 從rank中取出排名數據，沒有數據的給予9999999999
3. 從rank中抓取後方種類，定義為sub_catecory
4. 檢查title，若為空則補上空字串

- 數據分析
1. sub_category中 Beauty & Personal Care與None是相對較多購買量的商品種類
2. 查閱近一年每個月的販售量，逐年下降(有點糟...
3. 價格分群 (發現太貴的比較少人買，所以濾掉最貴的那群)
4. TF-IDF前處理，將標點符號、介詞和出現次數少次的詞等雜訊刪除
5. TF-IDF分析(之前學員的分享幫助很大，又加入了介係詞的過濾與kernel的finetune)，可以找到某群特別命中率特高 
6. 評分5星者具有較高的購買機會
7. 抓取訓練資料時，其時間越靠近測試資料相對越準確

- 圖像分析
1. 利用keras.applications中的DenseNet121 pre-train model(其總分類共有1000類)
2. 使用DenseNet121的原因，判斷種類多；雖然準確度較低，但是速度相對快
3. 利用metadata中的"imageURLHighRes"抓取圖片，並resize成model input size進行預測，並取其預測值最高者的index(0-999)
4. 預測的信賴值過低(低於0.5)則定義種類為1000
5. 若"imageURLHighRes"為空，定義種類為1001
6. 最終在使用時，發現1001與1000的購買率都不高，所以直接刪除

- 小結
1. 若使用TD-IDF對title、describe作分析，雖然有刪除標點符號與介詞等雜訊，但是種類仍聚集在一類之中，若只抓此類做推薦，分數可達0.141
2. 使用圖片分析種類，可以有比較均勻的分布
3. 最終使用圖片分類、overall=5、價格太貴刪除、sub_category="Beauty & Personal Care"、抓取前年9月overall當對照組
  ==> 最終最佳分數為0.169

- collaborating分析
目前最佳的推薦系統(for新用戶)若只看舊用戶其分數為0.007

1. user-based推薦分數為0
2. item-based推薦分數為0.001694915254237288
3. surprise套件遇到ram不夠的問題
3. user-based with imgType推薦分數為0
4. item-based with imgType推薦分數為0.003389830508474576
使用策略為k//3由item-based with imgType，剩餘的由目前最佳的推薦系統(for新用戶)隨機抽取補齊
  ==> 因為random產生其最佳分數會介於0.171-0.166之間

- collaborating分析(補充)
1. 嘗試surprise SVD/SVDpp/NMF，測試在舊客戶的表現都為0
2. 嘗試LightFM(尚未完成)

- 補充
上述的分析，都用利用2018/08、2017/09這兩個月的資料當作驗證集驗證趨勢一致

- 仍可改善的方向
1. 字串分析最後沒時間嘗試與影像分析結合討論，未來有機會想嘗試看看
2. 字串分析中若可把同義字，做成同組分析，感覺可以更仔細的抓取資料
3. 影像分析此次是直接使用model，但是其實有很多商品是在1000類之外的，如果可以藉由model output vector做Kmean分析的話，可以更仔細的分類。
4. surprise/LightFM等分析結果不如預期，仍需要仔細研究
5. 看到很多學員的EDA分析，感覺還有很多可以學習的東西

=====================================================================================================
=====================================================================================================

# content_based_filter.ipynb (content base)

使用邏輯如下
- 資料清理

把matedata重複asin的資料刪除

從rank中取出排名數據，沒有數據的給予9999999999

從rank中抓取後方種類，定義為sub_catecory

檢查title，若為空則補上空字串

- 數據分析

sub_category中 Beauty & Personal Care與None是相對較多購買量的商品種類

查閱近一年每個月的販售量，逐年下降(有點糟...

價格分群 (發現太貴的比較少人買，所以濾掉最貴的那群)

TF-IDF分析(之前學員的分享幫助很大，又加入了介係詞的過濾與kernel的finetune)，可以找到某群特別命中率特高 

評分5星者具有較高的購買機會

抓取訓練資料時，其時間越靠近測試資料相對越準確

最後利用評價數量當作排序指標

 - 補充

上述的分析，都用利用2018/08、2017/09這兩個月的資料當作驗證集驗證趨勢一致

因為重複會員者人比較少，所以這次主要以新會員的取向做優化

=====================================================================================================
=====================================================================================================

# sample.ipynb (rule base)

- 新使用者分析:
1. 測試集中有沒有歷史資料的新客戶。[考量沒歷史資料者怎麼分析]
2. 即使有歷史資料，單人的購買數量基本不多
3. 訓練資料從2000-2018，測試資料只有2018-0901(一個月
4. 從訓練資料可看出2017-2018年，評分median為5分者，通常為熱銷款[***]
5. (無品牌|'TIGI')似乎受大家歡迎[***]

- 重複購買使用者分析:
1. 沒有人購買重複的東西
2. 相同品牌的購買率也不高(但至少有)[*]
3. 曾經看過的，完全沒購買


# 結論

若user為新
1. 抓取(近期+一年前夏秋)median5分商品
2. 抓起其中沒有品牌資訊+品牌為TIGI者

若user為舊
1. 抓取曾經買過的品牌物件，並去除曾經買過的商品 -> N
2. 10-N 件商品由 新user清單中隨機抓取

**最終分數=0.13898305084745763**

