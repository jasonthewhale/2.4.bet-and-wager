### 2.4.bet-and-wager
尽管为了一笑跟朋友玩玩石头、剪刀、布很有趣，但在代码中跟敌人和你所有赖以生存的东西玩会更加更有意思。让我们来改变我们的程序，以便Alice可以跟Bob下注，无论谁赢了都会带走锅。  
这一次，让我们以JavaScript[前端](https://docs.reach.sh/ref-model.html#%28tech._frontend%29)开始然后我们将会返回到Reach的代码并把新的方法连接起来。  
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

第10行展示了显示货币金额（最多4个小数位）的功能  
第11行展示了用于获得参与者的余额并将其显示（最多4个小数位）的功能  
第12和13行在游戏开始前为Alice和Bob获得余额  
接下来我们会更新Alice的用户界面来加入她的赌注  

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...  
32    [backend](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28backend%29%29%29).Alice(ctcAlice, {  
33      ...Player('Alice'),  
34      wager: [stdlib](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28stdlib%29%29%29).[parseCurrency](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28parse.Currency%29%29%29)(5),  
35    }),  
..    // ...  

第33行将正常的Player界面拼接到Alice的界面中   
第34行将她的赌注定义为网络中的5个[网络代币](https://docs.reach.sh/ref-model.html#%28tech._network._token%29)，这是在[参与者交互界面](https://docs.reach.sh/ref-programs-module.html#%28tech._participant._interact._interface%29)中使用具体值而不是函数的示例  

对于Bob,我们将会改变他的界面以显示赌注，并通过返回值立即接受赌注  

[tut-3/index.mjs](https://github.com/reach-sh/reach-lang/blob/master/examples/tut-3/index.mjs#L5-L13)  
..    // ...  
36    [backend](https://docs.reach.sh/ref-backend-js.html#%28javascript._%28%28backend%29%29%29).Bob(ctcBob, {  
37      ...Player('Bob'),  
38      acceptWager: (amt) => {  
39        console.log(`Bob accepts the wager of ${fmt(amt)}.`);   
40      },  
41    }),  
..    // ...  
