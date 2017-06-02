# multi</br>
My multitasking library for lua</br>
To install copy the multi folder into your enviroment and you are good to go</br>

It is a pure lua binding if you ingore the intergrations (WIP)</br>


Usage:</br>
```lua
--Basic usage
require("multi.all") -- gets the entire library
alarm=multi:newAlarm(3) -- in seconds can go to .001 uses the built in os.clock()
alarm:OnRing(function(a)
  print("3 Seconds have passed!")
  a:Reset(n) -- if n were nil it will reset back to 3, or it would reset to n seconds
end)
multi:mainloop() -- the main loop of the program, multi:umanager() exists as well to allow intergration in other loops Ex: love2d love.update function. More on this binding in the wiki!
```
The library is module so you only need to require what you need. Because of this the global enviroment is altered</br>

There are many useful objects that you can use</br>
Check out the wiki for detailed usage, but here are the objects:</br>
- Process#</br>
- Queuer#</br>
- Alarm</br>
- Loop</br>
- Event</br>
- Step</br>
- Range</br>
- TStep</br>
- TLoop</br>
- Condition</br>
- Connection</br>
- Timer</br>
- Updater</br>
- Thread*</br>
- Trigger**</br>
- Task</br>
- Job</br>
- Function</br>
- Watcher***</br>
#Both a process and queue act like the multi namespace, but allows for some cool things. Because they use the other objects an example on them will be done last</br>
*Uses the built in coroutine features of lua, these have an interesting interaction with the other means of multi-tasking</br>
**Triggers are kind of useless after the creation of the Connection</br>
***Watchers have no real purpose as well I made it just because.</br>

# Examples of each object being used</br>
We already showed alarms in action so lets move on to a Loop object

Throughout these examples I am going to do some strange things in order to show other features of the library!

# LOOPS
```lua
-- Loops
require("multi.all") -- gets the entire library
count=0
loop=multi:newLoop(function(dt,self) -- dt is delta time and self is a reference to itself
  count=count+1
  if count > 10 then
    self:Break() -- All methods on the multi objects are Proper case, where as methods on the multi or process/queuer namespace are camel case
    -- self:Break() will stop the loop and trigger the OnBreak(func) method
    -- Stopping is the act of Pausing and deactivating the object! All objects can have the multiobj:Break() command on it!
  else
    print("Loop #"..count.."!")
  end
end)
loop:OnBreak(function(self)
  print("You broke me :(")
end)
multi:mainloop()
```
# Output
Loop #1!</br>
Loop #2!</br>
Loop #3!</br>
Loop #4!</br>
Loop #5!</br>
Loop #6!</br>
Loop #7!</br>
Loop #8!</br>
Loop #9!</br>
Loop #10!</br>
You broke me :(</br>


With loops out of the way lets go down the line

This library aims to be Async like. In reality everything is still on one thread *unless you are using the lanes intergration module WIP* (More on that later)

# EVENTS
```lua
-- Events, these were the first objects introduced into the library. I seldomly use them in their pure form though, but later on you'll see their advance uses!
-- Events on there own don't really do much... We are going to need 2 objects at least to get something going
require("multi.all") -- gets the entire library
count=0
-- lets use the loop again to add to count!
loop=multi:newLoop(function(dt,self)
  count=count+1
end)
event=multi:newEvent(function() return count==100 end) -- set the event
event:OnEvent(function(self) -- connect to the event object
  loop:Pause() -- pauses the loop from running!
  print("Stopped that loop!")
end) -- events like alarms need to be reset the Reset() command works here as well
multi:mainloop()
```
# Output
Stopped that loop!

# STEPS
```lua
require("multi.all")
-- Steps, are like for loops but non blocking... You can run a loop to infintity and everything will still run I will combine Steps with Ranges in this example.
step1=multi:newStep(1,10,1,0) -- Some explaining is due. Argument 1 is the Start # Argument 2 is the ResetAt # (inclusive) Argument 3 is the count # (in our case we are counting by +1, this can be -1 but you need to adjust your start and resetAt numbers)
-- The 4th Argument is for skipping. This is useful for timing and for basic priority management. A priority management system is included!
step2=multi:newStep(10,1,-1,1) -- a second step, notice the slight changes!
step1:OnStart(function(self)
  print("Step Started!")
end)
step1:OnStep(function(pos,self)
  if pos<=10 then -- what what is this? the step only goes to 10!!!
    print("Stepping... "..pos)
   else
    print("How did I get here?")
   end
end)
step1:OnEnd(function(self)
  print("Done!")
  -- We finished here, but I feel like we could have reused this step in some way... Yeah I soule Reset() it, but what if i wanted to change it...
  if self.endAt==10 then -- lets only loop once
	self:Update(1,11,1,0) -- oh now we can reach that else condition!
  end
  -- Note Update() will restart the step!
end)

-- step2 is bored lets give it some love :P
step2.range=step2:newRange() -- Set up a range object to have a nested step in a sense! Each nest requires a new range
-- it is in your interest not to share ranges between objects! You can however do it if it suits your needs though
step2:OnStep(function(pos,self)
  -- for 1=1,math.huge do
    --  print("Haha I am holding the code up because I can!!!")
  --end
  -- We dont want to hold things up, but we want to nest.
  -- Note a range is not nessary if the nested for loop has a small range, if however the range is rather large you may want to allow other objects to do some work
  for i in self.range(1,100) do
    print(pos,i) -- Now our nested for loop is using a range object which allows for other objects to get some cpu time while this one is running
  end
end)
-- TSteps are just like alarms and steps mixed together, the only difference in construction is the 4th Argument. On a TStep that argument controls time. The defualt is 1
-- The Reset(n) works just like you would figure!
step3=multi:newTStep(1,10,.5,2) -- lets go from 1 to 10 counting by .5 every 2 seconds
step3:OnStep(function(pos,self)
  print("Ok "..pos.."!")
end)
multi:mainloop()
```
# Output

Note: the output on this one is huge!!! So I had to ... some parts! You need to run this for your self to see what is going on!</br>
Step Started!</br>
Stepping... 1</br></br>
10	1</br>
Stepping... 2</br>
10	2</br>
Stepping... 3</br>
10	3</br>
...</br>
Ok 9.5!</br>
Ok 10!</br>

# TLOOPS
```lua
require("multi.all")
-- TLoops are loops that run ever n second. We will also look at condition objects as well
-- Here we are going to modify the old loop to be a little different
count=0
loop=multi:newTLoop(function(self) -- We are only going to coult with this loop, but doing so using a condition!
  while self:condition(self.cond) do
    count=count+1
  end
  print("Count is "..count.."!")
  self:Destroy() -- Lets destroy this object, casting it to the dark abyss MUHAHAHA!!!
  -- the reference to this object will be a phantom object that does nothing!
end,1) -- Notice the ',1' after the function! This is where you put your time value!
loop.cond=multi:newCondition(function() return count<=100 end) -- conditions need a bit of work before i am happy with them
multi:mainloop()
```
# Output
Count is 101!

# Connections
These are my favorite objects and you'll see why. They are very useful objects for ASync connections!

```lua
require("multi.all")
-- Lets create the events
yawn={} -- ill just leave that there
OnCustomSafeEvent=multi:newConnection(true) -- lets pcall the calls incase something goes wrong defualt
OnCustomEvent=multi:newConnection(false) -- lets pcall the calls incase something goes wrong
OnCustomEvent:Bind(yawn) -- create the connection lookup data in yawn

-- Lets connect to them, a recent update adds a nice syntax to connect to these
cd1=OnCustomSafeEvent:Connect(function(arg1,arg2,...)
  print("CSE1",arg1,arg2,...)
end,"bob") -- lets give this connection a name
cd2=OnCustomSafeEvent:Connect(function(arg1,arg2,...)
  print("CSE2",arg1,arg2,...)
end,"joe") -- lets give this connection a name
cd3=OnCustomSafeEvent:Connect(function(arg1,arg2,...)
  print("CSE3",arg1,arg2,...)
end) -- lets not give this connection a name

-- no need for connect, but I kept that function because of backwards compatibility
OnCustomEvent(function(arg1,arg2,...)
  print(arg1,arg2,...)
end)

-- Now within some loop/other object you trigger the connection like
OnCustomEvent:Fire(1,2,"Hello!!!") -- fire all conections

-- You may have noticed that some events have names! See the following example!
OnCustomSafeEvent:getConnection("bob"):Fire(1,100,"Bye!") -- fire only bob!
OnCustomSafeEvent:getConnection("joe"):Fire(1,100,"Hello!") -- fire only joe!!
OnCustomSafeEvent:Fire(1,100,"Hi Ya Folks!!!") -- fire them all!!!

-- Connections have more to them than that though!
-- As seen above cd1-cd3 these are hooks to the connection object. This allows you to remove a connection
-- For Example:
cd1:Remove() -- remove this connection from the master connection object
print("------")
OnCustomSafeEvent:Fire(1,100,"Hi Ya Folks!!!") -- fire them all again!!!
-- To remove all connections use:
OnCustomSafeEvent:Remove()
print("------")
OnCustomSafeEvent:Fire(1,100,"Hi Ya Folks!!!") -- fire them all again!!!
```
# Output
1	2	Hello!!!</br>
CSE1	1	100	Bye!</br>
CSE2	1	100	Hello!</br>
CSE1	1	100	Hi Ya Folks!!!</br>
CSE2	1	100	Hi Ya Folks!!!</br>
CSE3	1	100	Hi Ya Folks!!!</br>
------</br>
CSE2	1	100	Hi Ya Folks!!!</br>
CSE3	1	100	Hi Ya Folks!!!</br>
------</br>

You may think timers should be bundled with alarms, but they are a bit different and have cool features</br>
# TIMERS
```lua
-- You see the thing is that all time based objects use timers eg. Alarms, TSteps, and Loops. Timers are more low level!
require("multi.all")
local clock = os.clock
function sleep(n)  -- seconds
  local t0 = clock()
  while clock() - t0 <= n do end
end -- we will use this later!

timer=multi:newTimer()
timer:Start()
-- lets do a mock alarm
set=3 -- 3 seconds
a=0
while timer:Get()<=set do
  -- waiting...
  a=a+1
end
print(set.." second(s) have passed!")
-- Timers can do one more thing that is interesting and that is pausing them!
timer:Pause()
print(timer:Get()) -- should be really close to 'set'
sleep(3)
print(timer:Get()) -- should be really close to 'set'
timer:Resume()
sleep(1)
print(timer:Get()) -- should be really close to the value of set + 1
timer:Pause()
print(timer:Get()) -- should be really close to 'set'
sleep(3)
print(timer:Get()) -- should be really close to 'set'
timer:Resume()
sleep(1)
print(timer:Get()) -- should be really close to the value of set + 2
```
# Output
Note: This will make more sense when you run it for your self</br>
3 second(s) have passed!</br>
3.001</br>
3.001</br>
4.002</br>
4.002</br>
4.002</br>
5.003</br>

# UPDATER
```lua
require("multi.all")
updater=multi:newUpdater(5) -- really simple, think of a look with the skip feature of a step
updater:OnUpdate(function(self)
  --print("updating...")
end)
-- Here every 5 steps the updater will do stuff!
-- But I feel it is now time to touch into priority management, so lets get into basic priority stuff and get into a more advance version of it
--[[
multi.Priority_Core -- Highest form of priority
multi.Priority_High
multi.Priority_Above_Normal
multi.Priority_Normal -- The defualt form of priority
multi.Priority_Below_Normal
multi.Priority_Low
multi.Priority_Idle -- Lowest form of priority

Note: These only take effect when you enable priority, otherwise everything is at a core like level!
We aren't going to use regular objects to test priority, but rather benchmarks!
to set priority on an object though you would do
multiobj:setPriority(one of the above)
]]
-- lets bench for 3 seconds using the 3 forms of priority! First no Priority
multi:benchMark(3,nil,"Regular Bench: "):OnBench(function() -- the onbench() allows us to do each bench after each other!
  print("P1\n---------------")
  multi:enablePriority()
  multi:benchMark(3,multi.Priority_Core,"Core:")
  multi:benchMark(3,multi.Priority_High,"High:")
  multi:benchMark(3,multi.Priority_Above_Normal,"Above_Normal:")
  multi:benchMark(3,multi.Priority_Normal,"Normal:")
  multi:benchMark(3,multi.Priority_Below_Normal,"Below_Normal:")
  multi:benchMark(3,multi.Priority_Low,"Low:")
  multi:benchMark(3,multi.Priority_Idle,"Idle:"):OnBench(function()
    print("P2\n---------------")
	-- Finally the 3rd form
    multi:enablePriority2()
    multi:benchMark(3,multi.Priority_Core,"Core:")
    multi:benchMark(3,multi.Priority_High,"High:")
    multi:benchMark(3,multi.Priority_Above_Normal,"Above_Normal:")
    multi:benchMark(3,multi.Priority_Normal,"Normal:")
    multi:benchMark(3,multi.Priority_Below_Normal,"Below_Normal:")
    multi:benchMark(3,multi.Priority_Low,"Low:")
    multi:benchMark(3,multi.Priority_Idle,"Idle:")
	  os.exit()
  end)
end)
multi:mainloop() -- Notice how the past few examples did not need this, well only actors need to be in a loop! More on this in the wiki.
```
# Output
Note: These numbers will vary drastically depending on your compiler and cpu power</br>
Regular Bench:  2094137 Steps in 3 second(s)!</br>
P1</br>
---------------</br>
Below_Normal: 236022 Steps in 3 second(s)!</br>
Normal: 314697 Steps in 3 second(s)!</br>
Above_Normal: 393372 Steps in 3 second(s)!</br>
High: 472047 Steps in 3 second(s)!</br>
Core: 550722 Steps in 3 second(s)!</br>
Low: 157348 Steps in 3 second(s)!</br>
Idle: 78674 Steps in 3 second(s)!</br>
P2</br>
---------------</br>
Core: 994664 Steps in 3 second(s)!</br>
High: 248666 Steps in 3 second(s)!</br>
Above_Normal: 62166 Steps in 3 second(s)!</br>
Normal: 15541 Steps in 3 second(s)!</br>
Below_Normal: 3885 Steps in 3 second(s)!</br>
Idle: 242 Steps in 3 second(s)!</br>
Low: 971 Steps in 3 second(s)!</br>

Notice: Even though I started each bench at the same time the order that they finished differed the order is likely to vary on your machine as well!</br>


# TODO
- Process#</br>
- Queuer#</br>
- Thread*</br>
- Trigger**</br>
- Task</br>
- Job</br>
- Function</br>
- Watcher***</br>
- Write the wiki stuff
- Test for unknown bugs
