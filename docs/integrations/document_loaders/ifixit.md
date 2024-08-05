---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/ifixit.ipynb
---

# iFixit

>[iFixit](https://www.ifixit.com) 是网络上最大的开放维修社区。该网站包含近100k的维修手册，200k个关于42k设备的问题与答案，所有数据均根据CC-BY-NC-SA 3.0许可。

这个加载器将允许你使用他们的开放API从 `iFixit` 下载维修指南的文本、问答和设备的维基。它对于与技术文档相关的上下文和关于 `iFixit` 数据库中设备问题的答案非常有用。


```python
from langchain_community.document_loaders import IFixitLoader
```


```python
loader = IFixitLoader("https://www.ifixit.com/Teardown/Banana+Teardown/811")
data = loader.load()
```


```python
data
```



```output
[Document(page_content="# Banana Teardown\nIn this teardown, we open a banana to see what's inside.  Yellow and delicious, but most importantly, yellow.\n\n\n###Tools Required:\n\n - Fingers\n\n - Teeth\n\n - Thumbs\n\n\n###Parts Required:\n\n - None\n\n\n## Step 1\nTake one banana from the bunch.\nDon't squeeze too hard!\n\n\n## Step 2\nHold the banana in your left hand and grip the stem between your right thumb and forefinger.\n\n\n## Step 3\nPull the stem downward until the peel splits.\n\n\n## Step 4\nInsert your thumbs into the split of the peel and pull the two sides apart.\nExpose the top of the banana.  It may be slightly squished from pulling on the stem, but this will not affect the flavor.\n\n\n## Step 5\nPull open the peel, starting from your original split, and opening it along the length of the banana.\n\n\n## Step 6\nRemove fruit from peel.\n\n\n## Step 7\nEat and enjoy!\nThis is where you'll need your teeth.\nDo not choke on banana!\n", lookup_str='', metadata={'source': 'https://www.ifixit.com/Teardown/Banana+Teardown/811', 'title': 'Banana Teardown'}, lookup_index=0)]
```



```python
loader = IFixitLoader(
    "https://www.ifixit.com/Answers/View/318583/My+iPhone+6+is+typing+and+opening+apps+by+itself"
)
data = loader.load()
```


```python
data
```



```output
[Document(page_content='# My iPhone 6 is typing and opening apps by itself\nmy iphone 6 is typing and opening apps by itself. How do i fix this. I just bought it last week.\nI restored as manufactures cleaned up the screen\nthe problem continues\n\n## 27 Answers\n\nFilter by: \n\nMost Helpful\nNewest\nOldest\n\n### Accepted Answer\nHi,\nWhere did you buy it? If you bought it from Apple or from an official retailer like Carphone warehouse etc. Then you\'ll have a year warranty and can get it replaced free.\nIf you bought it second hand, from a third part repair shop or online, then it may still have warranty, unless it is refurbished and has been repaired elsewhere.\nIf this is the case, it may be the screen that needs replacing to solve your issue.\nEither way, wherever you got it, it\'s best to return it and get a refund or a replacement device. :-)\n\n\n\n### Most Helpful Answer\nI had the same issues, screen freezing, opening apps by itself, selecting the screens and typing on it\'s own. I first suspected aliens and then ghosts and then hackers.\niPhone 6 is weak physically and tend to bend on pressure. And my phone had no case or cover.\nI took the phone to apple stores and they said sensors need to be replaced and possibly screen replacement as well. My phone is just 17 months old.\nHere is what I did two days ago and since then it is working like a charm..\nHold the phone in portrait (as if watching a movie). Twist it very very gently. do it few times.Rest the phone for 10 mins (put it on a flat surface). You can now notice those self typing things gone and screen getting stabilized.\nThen, reset the hardware (hold the power and home button till the screen goes off and comes back with apple logo). release the buttons when you see this.\nThen, connect to your laptop and log in to iTunes and reset your phone completely. (please take a back-up first).\nAnd your phone should be good to use again.\nWhat really happened here for me is that the sensors might have stuck to the screen and with mild twisting, they got disengaged/released.\nI posted this in Apple Community and the moderators deleted it, for the best reasons known to them.\nInstead of throwing away your phone (or selling cheaply), try this and you could be saving your phone.\nLet me know how it goes.\n\n\n\n### Other Answer\nIt was the charging cord! I bought a gas station braided cord and it was the culprit. Once I plugged my OEM cord into the phone the GHOSTS went away.\n\n\n\n### Other Answer\nI\'ve same issue that I just get resolved.  I first tried to restore it from iCloud back, however it was not a software issue or any virus issue, so after restore same problem continues. Then I get my phone to local area iphone repairing lab, and they detected that it is an LCD issue. LCD get out of order without any reason (It was neither hit or nor slipped, but LCD get out of order all and sudden, while using it) it started opening things at random. I get LCD replaced with new one, that cost me $80.00 in total  ($70.00 LCD charges + $10.00 as labor charges to fix it). iPhone is back to perfect mode now.  It was iphone 6s. Thanks.\n\n\n\n### Other Answer\nI was having the same issue with my 6 plus, I took it to a repair shop, they opened the phone, disconnected the three ribbons the screen has, blew up and cleaned the connectors and connected the screen again and it solved the issue… it’s hardware, not software.\n\n\n\n### Other Answer\nHey.\nJust had this problem now. As it turns out, you just need to plug in your phone. I use a case and when I took it off I noticed that there was a lot of dust and dirt around the areas that the case didn\'t cover. I shined a light in my ports and noticed they were filled with dust. Tomorrow I plan on using pressurized air to clean it out and the problem should be solved.  If you plug in your phone and unplug it and it stops the issue, I recommend cleaning your phone thoroughly.\n\n\n\n### Other Answer\nI simply changed the power supply and problem was gone. The block that plugs in the wall not the sub cord. The cord was fine but not the block.\n\n\n\n### Other Answer\nSomeone ask!  I purchased my iPhone 6s Plus for 1000 from at&t.  Before I touched it, I purchased a otter defender case.  I read where at&t said touch desease was due to dropping!  Bullshit!!  I am 56 I have never dropped it!! Looks brand new!  Never dropped or abused any way!  I have my original charger.  I am going to clean it and try everyone’s advice.  It really sucks!  I had 40,000,000 on my heart of Vegas slots!  I play every day.  I would be spinning and my fingers were no where max buttons and it would light up and switch to max.  It did it 3 times before I caught it light up by its self.  It sucks. Hope I can fix it!!!!\n\n\n\n### Other Answer\nNo answer, but same problem with iPhone 6 plus--random, self-generated jumping amongst apps and typing on its own--plus freezing regularly (aha--maybe that\'s what the "plus" in "6 plus" refers to?).  An Apple Genius recommended upgrading to iOS 11.3.1 from 11.2.2, to see if that fixed the trouble.  If it didn\'t, Apple will sell me a new phone for $168!  Of couese the OS upgrade didn\'t fix the problem.  Thanks for helping me figure out that it\'s most likely a hardware problem--which the "genius" probably knows too.\nI\'m getting ready to go Android.\n\n\n\n### Other Answer\nI experienced similar ghost touches.  Two weeks ago, I changed my iPhone 6 Plus shell (I had forced the phone into it because it’s pretty tight), and also put a new glass screen protector (the edges of the protector don’t stick to the screen, weird, so I brushed pressure on the edges at times to see if they may smooth out one day miraculously).  I’m not sure if I accidentally bend the phone when I installed the shell,  or, if I got a defective glass protector that messes up the touch sensor. Well, yesterday was the worse day, keeps dropping calls and ghost pressing keys for me when I was on a call.  I got fed up, so I removed the screen protector, and so far problems have not reoccurred yet. I’m crossing my fingers that problems indeed solved.\n\n\n\n### Other Answer\nthank you so much for this post! i was struggling doing the reset because i cannot type userids and passwords correctly because the iphone 6 plus i have kept on typing letters incorrectly. I have been doing it for a day until i come across this article. Very helpful! God bless you!!\n\n\n\n### Other Answer\nI just turned it off, and turned it back on.\n\n\n\n### Other Answer\nMy problem has not gone away completely but its better now i changed my charger and turned off prediction ....,,,now it rarely happens\n\n\n\n### Other Answer\nI tried all of the above. I then turned off my home cleaned it with isopropyl alcohol 90%. Then I baked it in my oven on warm for an hour and a half over foil. Took it out and set it cool completely on the glass top stove. Then I turned on and it worked.\n\n\n\n### Other Answer\nI think at& t should man up and fix your phone for free!  You pay a lot for a Apple they should back it.  I did the next 30 month payments and finally have it paid off in June.  My iPad sept.  Looking forward to a almost 100 drop in my phone bill!  Now this crap!!! Really\n\n\n\n### Other Answer\nIf your phone is JailBroken, suggest downloading a virus.  While all my symptoms were similar, there was indeed a virus/malware on the phone which allowed for remote control of my iphone (even while in lock mode).  My mistake for buying a third party iphone i suppose.  Anyway i have since had the phone restored to factory and everything is working as expected for now.  I will of course keep you posted if this changes.  Thanks to all for the helpful posts, really helped me narrow a few things down.\n\n\n\n### Other Answer\nWhen my phone was doing this, it ended up being the screen protector that i got from 5 below. I took it off and it stopped. I ordered more protectors from amazon and replaced it\n\n\n\n### Other Answer\niPhone 6 Plus first generation….I had the same issues as all above, apps opening by themselves, self typing, ultra sensitive screen, items jumping around all over….it even called someone on FaceTime twice by itself when I was not in the room…..I thought the phone was toast and i’d have to buy a new one took me a while to figure out but it was the extra cheap block plug I bought at a dollar store for convenience of an extra charging station when I move around the house from den to living room…..cord was fine but bought a new Apple brand block plug…no more problems works just fine now. This issue was a recent event so had to narrow things down to what had changed recently to my phone so I could figure it out.\nI even had the same problem on a laptop with documents opening up by themselves…..a laptop that was plugged in to the same wall plug as my phone charger with the dollar store block plug….until I changed the block plug.\n\n\n\n### Other Answer\nHad the problem: Inherited a 6s Plus from my wife. She had no problem with it.\nLooks like it was merely the cheap phone case I purchased on Amazon. It was either pinching the edges or torquing the screen/body of the phone. Problem solved.\n\n\n\n### Other Answer\nI bought my phone on march 6 and it was a brand new, but It sucks me uo because it freezing, shaking and control by itself. I went to the store where I bought this and I told them to replacr it, but they told me I have to pay it because Its about lcd issue. Please help me what other ways to fix it. Or should I try to remove the screen or should I follow your step above.\n\n\n\n### Other Answer\nI tried everything and it seems to come back to needing the original iPhone cable…or at least another 1 that would have come with another iPhone…not the $5 Store fast charging cables.  My original cable is pretty beat up - like most that I see - but I’ve been beaten up much MUCH less by sticking with its use!  I didn’t find that the casing/shell around it or not made any diff.\n\n\n\n### Other Answer\ngreat now I have to wait one more hour to reset my phone and while I was tryin to connect my phone to my computer the computer also restarted smh does anyone else knows how I can get my phone to work… my problem is I have a black dot on the bottom left of my screen an it wont allow me to touch a certain part of my screen unless I rotate my phone and I know the password but the first number is a 2 and it won\'t let me touch 1,2, or 3 so now I have to find a way to get rid of my password and all of a sudden my phone wants to touch stuff on its own which got my phone disabled many times to the point where I have to wait a whole hour and I really need to finish something on my phone today PLEASE HELPPPP\n\n\n\n### Other Answer\nIn my case , iphone 6 screen was faulty. I got it replaced at local repair shop, so far phone is working fine.\n\n\n\n### Other Answer\nthis problem in iphone 6 has many different scenarios and solutions, first try to reconnect the lcd screen to the motherboard again, if didnt solve, try to replace the lcd connector on the motherboard, if not solved, then remains two issues, lcd screen it self or touch IC. in my country some repair shops just change them all for almost 40$ since they dont want to troubleshoot one by one. readers of this comment also should know that partial screen not responding in other iphone models might also have an issue in LCD connector on the motherboard, specially if you lock/unlock screen and screen works again for sometime. lcd connectors gets disconnected lightly from the motherboard due to multiple falls and hits after sometime. best of luck for all\n\n\n\n### Other Answer\nI am facing the same issue whereby these ghost touches type and open apps , I am using an original Iphone cable , how to I fix this issue.\n\n\n\n### Other Answer\nThere were two issues with the phone I had troubles with. It was my dads and turns out he carried it in his pocket. The phone itself had a little bend in it as a result. A little pressure in the opposite direction helped the issue. But it also had a tiny crack in the screen which wasnt obvious, once we added a screen protector this fixed the issues entirely.\n\n\n\n### Other Answer\nI had the same problem with my 64Gb iPhone 6+. Tried a lot of things and eventually downloaded all my images and videos to my PC and restarted the phone - problem solved. Been working now for two days.', lookup_str='', metadata={'source': 'https://www.ifixit.com/Answers/View/318583/My+iPhone+6+is+typing+and+opening+apps+by+itself', 'title': 'My iPhone 6 is typing and opening apps by itself'}, lookup_index=0)]
```

```python
loader = IFixitLoader("https://www.ifixit.com/Device/Standard_iPad")
data = loader.load()
```


```python
data
```



```output
[Document(page_content="标准版iPad\n苹果公司制造的平板电脑标准版。\n== 背景信息 ==\n\n标准版iPad于2010年1月首次推出，是苹果公司的平板电脑标准版。到目前为止，标准版iPad已经经历了十代。\n\n== 附加信息 ==\n\n* [link|https://www.apple.com/ipad-select/|官方苹果产品页面]\n* [link|https://en.wikipedia.org/wiki/IPad#iPad|官方iPad维基百科]", lookup_str='', metadata={'source': 'https://www.ifixit.com/Device/Standard_iPad', 'title': '标准版iPad'}, lookup_index=0)]
```

## 使用 /suggest 搜索 iFixit

如果您正在寻找基于关键字或短语搜索 iFixit 的更通用方法，/suggest 端点将返回与搜索词相关的内容，然后加载器将从每个建议项加载内容，并准备和返回文档。

```python
data = IFixitLoader.load_suggestions("Banana")
```

```python
data
```

```output
[Document(page_content='Banana\n美味的水果。钾的良好来源。黄色。\n== 背景信息 ==\n\n这种广受欢迎的水果通常拼写错误，形状像手机，既能提供营养，又能成为减缓后面快速行驶车辆障碍的工具。通常也用作“疯狂”或“失常”的同义词。\n\n从植物学上讲，香蕉被视为一种浆果，尽管它不属于包含草莓和覆盆子的烹饪浆果类别。香蕉属于芭蕉属，起源于东南亚和澳大利亚。现在主要在南美和中美洲种植，香蕉在全球范围内广泛供应。由于香蕉树能够全年结出水果，它们在发展中国家被特别视为主食。\n\n香蕉可以轻松打开。只需通过撬动茎的顶部来去掉外面的黄色外壳。然后，用断裂的部分沿着两侧向下剥离，直到内部的果肉暴露出来。一旦外壳被去掉，就无法重新组合。\n\n== 技术规格 ==\n\n* 尺寸：根据母树的基因可变\n* 颜色：根据成熟度、地区和季节可变\n\n== 其他信息 ==\n\n[link|https://en.wikipedia.org/wiki/Banana|维基百科：香蕉]', lookup_str='', metadata={'source': 'https://www.ifixit.com/Device/Banana', 'title': 'Banana'}, lookup_index=0),
 Document(page_content="# 香蕉拆解\n在这个拆解中，我们打开一根香蕉，看看里面有什么。黄色且美味，但最重要的是，黄色。\n\n\n### 所需工具：\n\n - 手指\n\n - 牙齿\n\n - 拇指\n\n\n### 所需部件：\n\n - 无\n\n\n## 第一步\n从一串中取出一根香蕉。\n不要挤得太紧！\n\n\n## 第二步\n用左手握住香蕉，用右手的拇指和食指夹住茎部。\n\n\n## 第三步\n向下拉茎部，直到外皮裂开。\n\n\n## 第四步\n将拇指插入外皮的裂口，将两侧拉开。\n暴露出香蕉的顶部。由于拉动茎而可能略微变形，但这不会影响味道。\n\n\n## 第五步\n从原来的裂口开始拉开外皮，沿着香蕉的长度打开。\n\n\n## 第六步\n将果肉从外皮中取出。\n\n\n## 第七步\n吃掉并享受！\n这是你需要牙齿的地方。\n不要噎到香蕉！\n", lookup_str='', metadata={'source': 'https://www.ifixit.com/Teardown/Banana+Teardown/811', 'title': 'Banana Teardown'}, lookup_index=0)]
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)