
# Timer Generator

Logic to define and launch Timers from ObjectScript methods/routines. The Timers will be signal a defined process with a particular token each X milliseconds.

This functionality can be easily integrated in our logic using the %SYSTEM.Event framework in InterSystems IRIS. Since each Timer will be an event defined against a particular process, we can implement the logic of that process to act in a different way depending on the Token with which the process has been waken-up.

## Install

Just load an compile the class `OPNLib.IoT.Timer`. If you want to look at some examples, also load and compile the `OPNEx.IoT.Timer.*`classes. There you have several examples and approaches to leverage this functionality.

## How does it work?

The concept is pretty easy. A process can _*subscribe*_ signal (Tokens) to a Timer defining the time at which the Timer should come back to the process signaling with that Token. Once the process is waken-up, it reviews the Token and take the appropiate actions executing a pre-defined logic.

Let's show it with a very simple example:

```javascript
    Class OPNEx.IoT.Timer.BasicSample Extends %RegisteredObject
    {
        ClassMethod Test(pTimeOut as %Integer=20)
        {
            #dim tTimer,tStop as %Integer = 0
            #dim tStart as %Integer = $piece($h,",",2)
            #dim tEndMsg as %String = "##CLOSING"
            #dim tPeriodMillisec as %Integer = 1000
            #dim tToken as %String = "BASICTOKEN001"

            do $system.Event.Clear($JOB)
            set tTimer = ##class(OPNLib.IoT.Timer).Subscribe(.tTimer,$JOB,,tPeriodMillisec,tEndMsg)

            while (tTimer>0)&&''tStop
            {
                set tListOfData = $system.Event.WaitMsg()

                set tData = $List(tListOfData,2)
                //Here we could execute a task depending on the data/token
                write !,"Token received....["_$piece($h,",",2)_"]: "_tData

                if (tData[tEndMsg)||($p($h,",",2)-tStart) > pTimeOut)
                {
                    set tStop = 1
                }
            }
        }
    }
```

As you can see, you don't need to set anything up to start using Timers. Just call Subscribe and, if it doesn't exist(\*), a new *tTimer* will be assigned for that $JOB-Token. The just created timer will start signaling that $JOB inmediately each *tPeriodMillisec*.

(\*) If there are already Timers available with free slots, then that Timer will be taken to also serve this subscription

## Basic Actions/Methods

You can see all the doc of main methods within the source code in more detail if you're interested. Here you have below what you really need to work with this functionality

Method | Description
-------------|-----------------------
Start()| It initiates a new Timer. If succeeds, it will return the PID associated to the Timer 
Stop(pTimer)| It stops the pTimer and UnSubscribe all the signals that is serving (if any)
StopAll()| It stops all the Timers running on this system, unsubscribing all their signals assigned. It will prompt before proceeding.
Subscribe(pTimer,pSubscriber,pToken,pPeriod,pEndMsg)| It will subscribe to pTimer the pair pSubscriber-pToken with a wake-up pPeriod and a pEndMsg to signaling unsubscription
UnSubscribe(pTimer,pSubscriber,pToken)| Whatever argument not indicated when calling this method is interpreted as "ALL" \[Timers|Subscribers|Tokens]
GetTimerFree(.pSlots)| It returns a positive integer with the PID of the first timer with free slots and will update pSlots with the number of free slots available in that Timer
Timers(pVerbose)| Returns a LIST with all the Timers currently active in the system. If pVerbose = 1, then it displays the list to the output device

---

Have fun!