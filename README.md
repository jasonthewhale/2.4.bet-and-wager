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
