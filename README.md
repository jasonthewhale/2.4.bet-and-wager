## 2.4.赌博与下注
尽管为博一笑跟朋友玩“石头剪刀布”很有趣，但在代码中跟敌人和你所赖以生存的东西玩会更加更有意思。让我们来修改我们的程序，以便Alice可以跟Bob下注，无论谁赢了都会带走那口锅。  
这一次，让我们从JavaScript[前端](https://docs.reach.sh/ref-model.html#%28tech._frontend%29)开始，然后我们将会返回到Reach的代码中，并把新的方法连接起来。  
既然我们将要转移资金，因此我们将在游戏开始之前记录每个参与者的余额，以便我们更清楚地显示他们最终赢得了什么。我们将在帐户创建与合同部署之间添加此代码。

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...  
 5    const [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29) = await loadStdlib(); 
 6    const startingBalance = [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[parseCurrency](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28parse.Currency%29%29%29)(10);  
 7    const accAlice = await [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[newTestAccount](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28new.Test.Account%29%29%29)(startingBalance);  
 8    const accBob = await [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[newTestAccount](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28new.Test.Account%29%29%29)(startingBalance);  
 9      
10    const fmt = (x) => [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[formatCurrency](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28format.Currency%29%29%29)(x, 4);  
11    const getBalance = async (who) => fmt(await [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[balanceof](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28balance.Of%29%29%29)(who));  
12    const beforeAlice = await getBalance(accAlice);  
13    const beforeBob = await getBalance(accBob);  
..    // ...

-第10行展示了显示货币金额（最多4个小数位）的功能。  
-第11行展示了用于获得参与者的余额并将其显示（最多4个小数位）的功能。  
-第12和13行在游戏开始前为Alice和Bob获得余额。    
-接下来我们会更新Alice的用户界面来加入她的赌注。     

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...  
32    [backend](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28backend%29%29%29).Alice(ctcAlice, {  
33      ...Player('Alice'),  
34      wager: [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[parseCurrency](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28parse.Currency%29%29%29)(5),  
35    }),  
..    // ...  

第33行将正常的Player界面拼接到Alice的界面中。      
第34行将她的赌注定义为网络中的5个[网络代币](https://docs.reach.sh/ref-model.html#%28tech._network._token%29)，这是在[参与者交互界面](https://docs.reach.sh/ref-programs-module.html#%28tech._participant._interact._interface%29)中使用具体值而不是函数的示例。   

对于Bob,我们将会改变他的界面以显示赌注，并通过返回值立即接受赌注。    

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...  
36    [backend](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28backend%29%29%29).Bob(ctcBob, {  
37      ...Player('Bob'),  
38      acceptWager: (amt) => {  
39        console.log(`Bob accepts the wager of ${fmt(amt)}.`);   
40      },  
41    }),  
..    // ...  

第38行到第40行定义了接受赌注的功能。    
最后，在计算结束后，我们将会再次获取余额并展示一条总结结果的信息。  

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...    
44    const afterAlice = await getBalance(accAlice);  
45    const afterBob = await getBalance(accBob);  
46    
47    console.log(`Alice went from ${beforeAlice} to ${afterAlice}.`);  
48    console.log(`Bob went from ${beforeBob} to ${afterBob}.`);  
..    // ...    

第44和第45行在结束后获得余额。    
第47和第48行打印出结果。     

对前端的这些更改仅处理展示和接口的问题，下注和转移资金的实际操作逻辑将在Reach的代码中进行。    
让我们来看看那些代码。   
首先，我们需要更新参与者交互界面。   

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)   
 1    'reach 0.1';  
 2    
 3    [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) Player =  
 4          { getHand: [Fun](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.Fun%29%29%29)([], [UInt](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.U.Int%29%29%29)),  
 5            seeOutcome: [Fun](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.Fun%29%29%29)([[UInt](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.U.Int%29%29%29)], [Null](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.Null%29%29%29)) };  
 6    [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) Alice =  
 7          { ...Player,  
 8            wager: [UInt](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.U.Int%29%29%29) };  
 9    [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) Bob =  
10          { ...Player,  
11            acceptWager: [Fun](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.Fun%29%29%29)([[UInt](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.U.Int%29%29%29)], [Null](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28.Null%29%29%29)) };    
12    
13    [export](https://docs.reach.sh/ref-programs-module.html#%28reach._%28%28export%29%29%29) [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) main =  
14      [Reach](https://docs.reach.sh/ref-programs-module.html#%28reach._%28%28.Reach%29%29%29).[App](https://docs.reach.sh/ref-programs-module.html#%28reach._%28%28.App%29%29%29)(  
15        {},  
16        [[Participant](https://docs.reach.sh/ref-programs-module.html#%28reach._%28%28.Participant%29%29%29)('Alice', Alice), [Participant](https://docs.reach.sh/ref-programs-module.html#%28reach._%28%28.Participant%29%29%29)('Alice', Alice)('Bob', Bob)],  
17        (A, B)  [=>](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3d._~3e%29%29%29) {  
..          // ...  
42          [exit](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28exit%29%29%29)(); });  

第6到第8行定义了Alice的界面为玩家界面，再加上一个叫做下注的整数值。     
第9到11行对Bob做了同样的工作，他有一个叫做接受赌注的方法来查看下注值。    
第16行将这些接口和相应的参与者相关联。这行代码的格式是[参与者构造函数](https://docs.reach.sh/ref-programs-module.html#%28tech._participant._constructor%29)的[元组](https://docs.reach.sh/ref-programs-compute.html#%28tech._tuple%29),其中第一个参数是一个名称为[后端](https://docs.reach.sh/ref-model.html#%28tech._backend%29)[参与者](https://docs.reach.sh/ref-model.html#%28tech._participant%29)的字符串,而第二个参数是[参与者交互接口](https://docs.reach.sh/ref-programs-module.html#%28tech._participant._interact._interface%29)，习惯上用类似的名字来命名它们  

该应用程序的三个部分中的每一个都必须进行更新以处理赌注。让我们先看看Alice的第一步。    

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)   
..    // ...  
18    A.[only](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28only%29%29%29)(() [=>](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3d._~3e%29%29%29) {  
19      [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) wager = [declassify](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29)([interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29).wager);  
20      [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) handA = [declassify](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29)([interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29).getHand()); });  
21    A.[publish](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28publish%29%29%29)(wager, handA)  
22      .[pay](https://docs.reach.sh/tut-3.html)(wager);  
23    [commit](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28commit%29%29%29)();  
..    // ...  

第19行让Alice将赌注[解密](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29)来进行传输  
第21行更新以便Alice与Bob分享赌注的金额    
第22行让她转移该金额作为她[广播](https://docs.reach.sh/ref-model.html#%28tech._publication%29)的一部分。如果下注未出现在21行，而出现在22行,则Reach编译器将引发异常.修改程序并尝试这个。这是因为[共识网络](https://docs.reach.sh/ref-model.html#%28tech._consensus._network%29)需要能够验证Alice[广播](https://docs.reach.sh/ref-model.html#%28tech._publication%29)中包含的[网络代币](https://docs.reach.sh/ref-model.html#%28tech._network._token%29)的数量，是否与[共识网络](https://docs.reach.sh/ref-model.html#%28tech._consensus._network%29)可获得的某些计算相匹配。   

接下来，Bob需要被展示赌注并给予接受赌注或转移资产的机会。    
[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)   
..    // ...  
25    B.[only](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28only%29%29%29)(() => {  
26      [interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29).acceptWager(wager);  
27      [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) handB = [declassify](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29)([interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29).getHand()); });  
28    B.[publish](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28publish%29%29%29)(handB)  
29      .[pay](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28pay%29%29%29)(wager);  
..    // ...  

第26行使Bob接受赌注。如果他不喜欢这些条款，那么他的前端可以不回复这个方法并且[DApp](https://docs.reach.sh/ref-model.html#%28tech._dapp%29)将会停止。    
第29行也让Bob进行下注。    

[DApp](https://docs.reach.sh/ref-model.html#%28tech._dapp%29)正在[共识步骤](https://docs.reach.sh/ref-model.html#%28tech._consensus._step%29)中运行，并且合约本身现在拥有下注金额的两倍。之前，它将计算结果，然后提交状态。但是现在，它需要查看结果并使用它来平衡帐户。  

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)   
..    // ...
31    [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) outcome = (handA [+](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28%2B%29%29%29) (4 - handB)) [%](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~25%29%29%29) 3;  
32    [const](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28const%29%29%29) outcome = (handA + (4 - handB) [forA, forB] =  
33          outcome [==](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3d~3d%29%29%29) 2 [?](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3f%29%29%29) [2, 0] :  
34          outcome [==](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3d~3d%29%29%29) 0 [?](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28~3f%29%29%29) [0, 2] :  
35          [1, 1];  
36    [transfer](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28transfer%29%29%29)(forA [*](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28%2A%29%29%29) wager).to(A);  
37    [transfer](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28transfer%29%29%29)(forA [*](https://docs.reach.sh/ref-programs-compute.html#%28reach._%28%28%2A%29%29%29) wager)(forB * wager).to(B);   
38    [commit](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28commit%29%29%29)();  
..    // ...

第33行到第35行通过确定每一方获得下注金额的数量，来根据结果计算给每个参与者的金额。如果结果为2，则Alice获胜，那么她将获得两份金额；如果为0，则Bob获胜，那么他将获得两份金额；否则，他们每个人都会得到一份金额。    
第36和37行用来转移相应的金额。这个转账是从合约到参与者的转移，而不是从参与者到彼此的转移，因为所有资金都位于合约内部。  
第38行提交应用程序的状态并允许参与者查看结果并执行。  

现在，我们可以运行这个程序，并通过运行来查看它的输出。  

 $ ./reach run  

Since the players act randomly, the results will be different every time. When I ran the program three times, this is the output I got:  

$ ./reach run  

Alice played Paper  

Bob accepts the wager of 5.  

Bob played Rock  

Alice saw outcome Alice wins  

Bob saw outcome Alice wins  

Alice went from 10 to 14.9999.  

Bob went from 10 to 4.9999.  

 

$ ./reach run  

Alice played Paper  

Bob accepts the wager of 5.  

Bob played Scissors  

Alice saw outcome Bob wins  

Bob saw outcome Bob wins  

Alice went from 10 to 4.9999.  

Bob went from 10 to 14.9999.  

 

$ ./reach run   

Alice played Rock  

Bob accepts the wager of 5.  

Bob played Scissors  

Alice saw outcome Alice wins  

Bob saw outcome Alice wins  

Alice went from 10 to 14.9999.  

Bob went from 10 to 4.9999.  

Alice和Bob的余额怎么会每次都变回10呢？这是因为每次我们运行“./reach run”时，它开始了一个测试网络的全新实例并为每个玩家注册账号。   

余额怎么会不是刚好为10，15和5呢？这是因为以太坊的交易消耗燃气费来运行。  

如果我们已经展示了所有的小数点，他们会看起来像这样：  
—  

Alice went from 10 to 14.999999999999687163.  

Bob went from 10 to 4.999999999999978229.  

...  
 
Alice went from 10 to 4.999999999999687163.  

Bob went from 10 to 14.999999999999978246.  

—  
为什么当Alice获胜时，赢得的钱会略微少于Bob呢？她不得不付钱来[部署](https://docs.reach.sh/ref-model.html#%28tech._deploy%29)这张合约，因为她在前端 有关[部署的指南部分](https://docs.reach.sh/guide-deploymode.html）讨论了如何避免这种差值。  

Alice做的还不错，如果她持续赌下去，她将在“石头剪刀布”中赚上一笔  

如果你的版本不能工作，请查看完整版本的[ tut-3/index.rsh](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.rsh)和[ tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs)，以确保你正确地复制了所有东西。  

既然有一个理由玩这个游戏，那结果证明存在一个重大的安全隐患。我们将会在[下一步](https://docs.reach.sh/tut-4.html)修复它；确保你不要用这个版本发布，不然Alice就要破产了！  

检查你的理解：Reach程序如何管理代币？  
1.他们不管理，你需要与Reach程序显式地共同管理代币；  
2.[pay](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28pay%29%29%29)的原始语句可以被添加到[publish](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28publish%29%29%29)原始语句中来发送资金给Reach程序，这样到时候就可以用[transfer](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28transfer%29%29%29)原始语句来把资金发送回参与者和其他地址。  
答案：  
2；[pay](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28pay%29%29%29)和[transfer](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28transfer%29%29%29)原始语句帮你处理这一切。
