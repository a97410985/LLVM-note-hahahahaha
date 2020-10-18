# LLVM tutorial

1. 如何安裝LLVM到windows電腦
   1. 使用visual studio開發且使用vcpkg方便安裝LLVM
      1. 到[vcpkg官網](https://github.com/eliben/llvm-clang-samples/blob/master/src_clang/matchers_rewriter.cpp)下載原始檔編譯
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018214735415.png" alt="image-20201018214735415" style="zoom:67%;" />
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018214803189.png" alt="image-20201018214803189" style="zoom:67%;" />
         從解壓縮的資料夾開啟cmd，然後打下面的指令，執行一個編譯原始碼的**批次檔**，就安裝了
         
         ```shell
         .\vcpkg\bootstrap-vcpkg.bat
         ```
         
         為了可以不用每次使用vcpkg.exe都輸入完整的路徑，要設環境變數，告訴cmd可以從哪裡找到檔案
         
         > 懶得輸入完整執行檔路徑的解決方法有cmd會自動幫你搜尋所有地方，但是這樣太浪費時間和資源，因此可以設定環境變數告訴cmd只需要找那些地方，這就是環境變數的緣由
         
         先到本機按下滑鼠右鍵再點擊內容
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018215303480.png" alt="image-20201018215303480" style="zoom:67%;" />
         然後按**進階系統設定**，再按跳出視窗的**環境變數**
         
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018215344583.png" alt="image-20201018215344583" style="zoom:67%;" />
         
         選取**Path**按編輯按鈕
         
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018215637883.png" alt="image-20201018215637883" style="zoom:67%;" />
         
         新增一條路徑為剛才安裝vcpkg的路徑
         ![image-20201018215713946](D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018215713946.png)
         
         這樣一來就可以直接打vcpkg，不需要打**D:\vcpkg\vcpkg**
         
         用search指令搜尋llvm如下
         
         ```shell
         vcpkg search llvm
         ```
         
         會得到
         <img src="D:\all md note\LLVM-note-hahahahaha\LLVM tutorial.assets\image-20201018215959316.png" alt="image-20201018215959316" style="zoom:67%;" />
         可以看到有列出**llvm**
         
         因此我們可以直接爽爽安裝llvm，下下面指令
         
         ```shell
         vcpkg install llvm
         ```
         
         然後大概等4小時llvm就安裝好了= =，其實可以只安裝部分的llvm，這裡不贅述，因為llvm後端對應到非常多不同的CPU架構，所以會很龐大，一般不需要那麼多。
         
         安裝好後要下integrate install，這樣才會跟IDE做**自動連結**，直接可以include標頭檔，**vcpkg**最方便的地方
         
         ```shell
         vcpkg integrate install
         ```
   
2. clang for AST(abstract syntax tree) match and rewrite

3. 