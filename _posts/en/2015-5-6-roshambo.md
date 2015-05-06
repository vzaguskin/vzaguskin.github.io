---
layout: post
title: AI for roshambo!
intname: roshambo1
---
## Implementing RoShamBo AI in python  ##

There is a Nash equilibrium for RoShamBo game - just make your moves random and you can't statistically lose. But many people and bots in competitions don't play random. And that means we can try beating them if we exploit their non-randomness. 

Let's make a bot for [http://www.rpscontest.com](http://www.rpscontest.com). You can play with it [here](http://www.rpscontest.com/human/5695872079757312?) or read the complete source code [there](https://github.com/vzaguskin/roshambo/blob/master/rpsbotvzv4.py).

The tournament has a simple API - you get the last move of your opponent via *input* global variable and make your move via *output* in the form of 'R', 'P' or 'S'. If the input is empty string, that's the first round and you can use that for initialisation. So, in the global scope we have:

```python

    if input == '':
    	tp = treePredictor()
    	output = tp.predict()    
    else:
    	tp.store(input)
    	output = tp.predict()

```
Let's assume that the opponent move depends on the result of his previous move. For example, if he played 'R'(*rock*) and lost to *paper* - next time he'll prefer to play *scissors* to beat the paper, and if he wins - he'll repeat *stone*. Other players might prefer to change the move after the victory, but repeat the same after tie.


To make the accounting simplier we need a concept of *roll*. *Roll* for 0 transforms *rock* to *rock*, *paper* to *paper*, etc. *Roll* to 1 transforms *rock* to *paper*, *paper* to *scissors*, *scissors* to *rock*. And roll to 2 transforms *rock* to *scissors*, *paper* to *rock* and *scissors* to *paper*. Interestingly, roll 1 for any move gives you what beats that move, and roll 2 - what is beaten by that move. 

So, we have 3 results(win, lose, tie) and 3 rolls and we want to count how often the opponents next move is a result of each roll from his previous move after each result. That gives us a 3*3 matrix of integer counts.

Here is how the method **store** would look:

```python

    def store(self, c):
        
        i1 = self.choices.index(c)
        
        if not (self.prevchoice is None or self.prevres is None):
            roll = self.getRollbyInd(self.prevchoice, i1)
            for i in range(3):
                for j in range(3):
                    self.dataarr[i][j]*=0.9
            self.dataarr[self.prevres][roll]+=1
            
            
        self.prevchoice = i1
        self.prevres = self.gameRes(c,self.prevmove)

```

**self.dataarr** - is our 3*3 statistics array.

Note that on each iteration we multiply the whole array to 0.9 - that discounts the older statistics relatively to more recent moves so we adopt to strategy change quicker.


Then we have to predict the opponent move based on the statistics. We'll make a weighted random choice of the predicted move where weights are the numbers of time the opponent made each roll to decide a next move. And make a move that would beat the move we've predicted. 


```python

    def predict(self):
        if self.prevchoice is None or self.prevres is None:
            ret= self.choices[randint(0,2)]
            self.prevmove = ret
            return ret
        arr=self.dataarr[self.prevres]
        
        predictedroll = weighted_choice([0,1,2], arr)
        predictedchoice = self.rollInd(self.prevchoice, predictedroll)
        
        choice = self.beatMat[predictedchoice]

        self.prevmove = self.choices[choice]
        return self.choices[choice]

```


The bot is currently going relatively well in the tournament with a winrate of ~75% and I cannot easily beat it myself. There are bots that beat him in the tournament though - but still the result is interesting enough as we just use one very simple strategy. Another important thing - if the opponents moves doesn't really statistically depend on his previous moves results, our counts have almost equal values and we essentially play random, so our play cannot be exploited and we guarantee a statistical tie. 

  