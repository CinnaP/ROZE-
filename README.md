# ROZE-

#!/usr/bin/env python
# encoding: utf-8
from pyo import *
import random
import wx
import os


s = Server().boot()
s.start()

"""
if "ROZE.app" in os.getcwd():
    PATH = os.path.join(os.getcwd)
else:
    PATH = os.path.join(os.getcwd, "Resources") 
""" 
 
NAME = "ROZE"
VERSION = "0.0.1"



class Chaos:
    def __init__(self, pitch=.5, detune=.001, balance=1, movement=0, volume=0, send=1):
        self.pitch = Sig(pitch)
        self.detune = Sig(detune)
        self.balance = SigTo(balance)
        self.movement = Sig(movement)
        self.volume = SigTo(volume)
        self.send = SigTo(send)
        self.lfo = LFO(freq=Randi(.4, 4, freq=1), type=6).range(.1, .99)
        self.rossler = Rossler(pitch=[(self.pitch - self.detune), self.pitch, (self.pitch + self.detune)],  
                               chaos=self.lfo*self.movement, stereo=True, mul=.7*self.balance*self.volume*self.send)
        self.lorenz = Lorenz(pitch=[(self.pitch - self.detune), self.pitch, (self.pitch + self.detune)],
                             chaos=self.lfo*self.movement, stereo=True, mul=.7*(1-self.balance)*self.volume*self.send)
        self.masterOut = DCBlock(self.rossler + self.lorenz)
        
    def out(self):
        self.masterOut.out()
        return self
        
    def getOut(self):
        return self.masterOut
        
    def setChaosVolume(self, x):
        self.volume.value = x
        
    def setChaosSend(self, x):
        self.send.value = x
        
    def setPitch(self, x):
        self.pitch.value = [(x - self.detune.value), x, (x + self.detune.value)]

    def setDetune(self, x):
        self.detune.value = x
        
    def setBalance(self, x):
        self.balance.value = x
    
    def setMovement(self, x):
        self.movement.value = x
        

class Boscavo:
    def __init__(self, time=.250, hats=.2, accents=3, volume=0, send=1):
        self.time = Sig(time)
        self.hats = Sig(hats)
        self.accents = Sig(accents)
        self.volume = SigTo(volume)
        self.send = SigTo(send)
        self.table = table = CosTable([(0,0), (10, 1), (100, .25), (1000, 0)])
        self.masterBeat = Euclide(time=self.time, onsets=1, taps=1).play()
        self.trigJitter = TrigRand(self.masterBeat, min=.3, max=self.accents)
        self.trigTable = TrigEnv(self.masterBeat, table, dur=self.trigJitter)
        self.trigVol1 = TrigRand(self.masterBeat, 0, .9)
        self.trigPitchJitter = TrigRand(self.masterBeat, .95, 1.05)
        self.trigHats = TrigRand(self.masterBeat, 0, self.hats)
        self.sloop = SineLoop(freq=[random.uniform(50, 60) for i in range(10)], feedback=self.trigHats, mul=self.trigTable*self.trigVol1*0.5*self.volume*self.send)
        self.reson1 = AllpassWG(input=self.sloop.mix(2), freq=55, feed=.5, mul=.1*self.volume*self.send)
        self.reson2 = AllpassWG(input=self.sloop.mix(2), freq=100, feed=.3, mul=.1*self.volume*self.send)
        self.masterOut = DCBlock(self.sloop + self.reson1 + self.reson2)
        
    def out(self):
        self.masterOut.out()s
        return self
        
    def getOut(self):
        return self.masterOut
        
    def setBoscavoVolume(self, x):
        self.volume.value = x
        
    def setBoscavoSend(self, x):
        self.send.value = x

    def setTime(self, x):
        self.time.value = x
        
    def setHats(self, x):
        self.hats.value = x
        
    def setAccents(self, x):
        self.accents.value = x
        
    def setOnsets(self, x):
        self.masterBeat.onsets = x
        
    def setTaps(self, x):
        self.masterBeat.taps = x
        
    def stop(self):
        self.masterBeat.stop()
        
    def play(self):
        self.masterBeat.play()
        
 
class Elarc:
    def __init__(self, harms=40, freq=100, feed=.9, volume=0, send=1):
        self.harms = Sig(harms)
        self.freq = Sig(freq)
        self.feed = Sig(feed)
        self.volume = SigTo(volume)
        self.send = SigTo(send)
        self.blit = Blit(freq=[random.uniform(1, 10) for i in range(3)], 
                         harms=self.harms, mul=.3*self.volume*self.send)
        self.reson = AllpassWG(input=self.blit, freq=self.freq, feed=self.feed, mul=.3*self.volume*self.send)
        self.delay = SDelay(input=self.reson)
        self.dist = Disto(input=self.delay, drive=.95, mul=.3*self.volume*self.send)
        self.masterOut = DCBlock(self.blit + self.reson + self.dist)
        
    def out(self):
        self.masterOut.out()
        return self
        
    def getOut(self):
        return self.masterOut
        
    def setElarcVolume(self, x):
        self.volume.value = x
        
    def setElarcSend(self, x):
        self.send.value = x
        
    def setHarms(self, x):
        self.harms.value = x
        self.harms.get()

    def setFreq(self, x):
        self.freq.value = x

    def setFeed(self, x):
        self.feed.value = x
    
    def lowFlow(self):
        self.blit.freq=[random.uniform(2, 3) for i in range(3)]
    
    def fastFlow(self):
        self.blit.freq=[random.uniform(1, 10) for i in range(3)]
   
     
class Slift:
    def __init__(self, white=1, tone=500,  time=.250, dur=.01, delay=0, cutoff=10000, volume=0, send=1):
        self.white = Sig(white)
        self.tone = SigTo(tone)
        self.cutoff = Sig(cutoff)
        self.time = Sig(time)
        self.dur = Sig(dur)
        self.delay = Sig(delay)
        self.cutoff = Sig(cutoff)
        self.volume = SigTo(volume)
        self.send = SigTo(send)
        self.table = CosTable([(0,0), (100, 1), (200, .25), (8191, 0)])
        self.beat = Metro(time=self.time, poly=1).play()
        self.trig = TrigEnv(self.beat, table=self.table, dur=self.dur, mul=.3)
        self.lrz = Lorenz(self.white, 1, True, 0.49, 0.5)
        self.sloop = SineLoop([self.tone * .5, self.tone, self.tone * 2], feedback=self.lrz, mul=self.trig*self.volume*.4*self.send)
        self.fltr = ButLP(input=self.sloop, freq=self.cutoff)
        self.dly = Delay(input=self.fltr, delay=self.delay, feedback=.6, mul=self.volume*self.send)
        self.span = SPan(input=self.dly, pan=Randi(0, 1, 1))
        self.masterOut = DCBlock(self.fltr + self.span)
        
    def out(self):
        self.masterOut.out()
        return self
        
    def getOut(self):
        return self.masterOut
        
    def setSliftVolume(self, x):
        self.volume.value = x
        
    def setSliftSend(self, x):
        self.send.value = x
        
    def setWhite(self, x):
        self.white.value = x
        
    def setTone(self, x):
        self.tone.value = x
        
    def setTime(self, x):
        self.time.value = x
        
    def setDur(self, x):
        self.dur.value = x
        
    def setDelay(self, x):
        self.delay.value = x
    
    def setCutoff(self, x):
        self.cutoff.value = x
        
    def stop(self):
        self.beat.stop()
        
    def play(self):
        self.beat.play()
        
       
class Zone1:
    def __init__ (self, input, zone1=.5, decay=3):
        self.zone1 = SigTo(zone1)
        self.decay = Sig(decay)
        self.reverb1 = STRev(input=input,inpos=Randi(0, 1, .15), revtime=self.decay, mul=self.zone1)
        self.fltr = ButHP(input=input, freq=1000)
        self.reverb2 = STRev(input=self.fltr,inpos=Randi(0, 1, 1), bal=1, revtime=self.decay, mul=self.zone1*.1)
       
    def out(self):
        self.reverb1.out()
        self.reverb2.out()
        return self
        
    def setZone1(self, x):
        self.zone1.value = x

    def setDecay(self, x):
        self.decay.value = x
        
        
class Zone2:
    def __init__ (self, input, zone2=.5, plate=100):
        self.zone2 = SigTo(zone2)
        self.plate = SigTo(plate)
        self.reson = Waveguide(input=input, freq=self.plate, dur=1, mul=self.zone2*.5)
        
    def out(self):
        self.reson.out()
        return self
        
    def setZone2(self, x):
        self.zone2.value = x
        
    def setPlate(self, x):
        self.plate.value = x
  
      
class Zone3:
    def __init__ (self, input, zone3=.5):
        self.zone3 = SigTo(zone3)
        self.delay = Delay(input=input, delay=.3, feedback=.3, mul=self.zone3)
        
    def out(self):
        self.delay.out()
        return self
        
    def setZone3(self, x):
        self.zone3.value = x

        

        
 
        
    

########################################################################################

cha = Chaos().out()
chaSend1 = Chaos(send=0)
chaSend2 = Chaos(send=0)
chaSend3 = Chaos(send=0)

bos = Boscavo().out()
bosSend1 = Boscavo(send=0)
bosSend2 = Boscavo(send=0)
bosSend3 = Boscavo(send=0)

ela = Elarc().out()
elaSend1 = Elarc(send=0)
elaSend2 = Elarc(send=0)
elaSend3 = Elarc(send=0)

sli = Slift().out()
sliSend1 = Slift(send=0)
sliSend2 = Slift(send=0)
sliSend3 = Slift(send=0)

zo1 = Zone1(chaSend1.getOut() + elaSend1.getOut() + bosSend1.getOut() + sliSend1.getOut()).out()

zo2 = Zone2(chaSend2.getOut() + elaSend2.getOut() + bosSend2.getOut() + sliSend2.getOut()).out()

zo3 = Zone3(chaSend3.getOut() + elaSend3.getOut() + bosSend3.getOut() + sliSend3.getOut()).out()

clock = .5

def setClock(x):
    global clock
    clock = x
    bos.setTime(clock)
    bosSend1.setTime(clock)
    bosSend2.setTime(clock)
    bosSend3.setTime(clock)
    sli.setTime(clock)
    sliSend1.setTime(clock)
    sliSend2.setTime(clock)
    sliSend3.setTime(clock)
    

#s.gui(locals())

########################################################################################



class MyFrame(wx.Frame):
    
    def __init__(self, parent, title, pos, size):

### UTILITY --------------

        # The Frame
        wx.Frame.__init__(self, parent, id=-1, title=title, pos=pos, size=size)
        self.panel = wx.Panel(self)
        self.panel.SetBackgroundColour("#FD83EC")
        
        
### CHAOS --------------
        
        # The Volume Slider
        self.chaos_volume = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(60, 82), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.chaos_volume.Bind(wx.EVT_SLIDER, self.changeChaosVolume)

        # The Pitch Slider
        self.pitchText = wx.StaticText(self.panel, id=-1, label="Pitch", pos=(140, 60), size=wx.DefaultSize)
        self.pitch = wx.Slider(self.panel, id=-1, value=50, minValue=15, maxValue=100, pos=(140, 80), size=(250, -1))
        self.pitch.Bind(wx.EVT_SLIDER, self.changePitch)
        
        # The Detune Slider
        self.detuneText = wx.StaticText(self.panel, id=-1, label="Detune", pos=(140, 125), size=wx.DefaultSize)
        self.detune = wx.Slider(self.panel, id=-1, value=1, minValue=1, maxValue=100, pos=(140, 145), size=(250, -1))
        self.detune.Bind(wx.EVT_SLIDER, self.changeDetune)

        # The Balance Slider
        self.balanceText = wx.StaticText(self.panel, id=-1, label="Balance", pos=(140, 190), size=wx.DefaultSize)
        self.balance = wx.Slider(self.panel, id=-1, value=100, minValue=0, maxValue=100, pos=(140, 210), size=(250, -1))
        self.balance.Bind(wx.EVT_SLIDER, self.changeBalance)
        
        # The Movement Slider
        self.movementText = wx.StaticText(self.panel, id=-1, label="Movement", pos=(435, 145), size=wx.DefaultSize)
        self.movement = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(415, 82), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.movement.Bind(wx.EVT_SLIDER, self.changeMovement)
        

### BOSCAVO --------------
        
        # The Volume Slider
        self.boscavo_volume = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(60, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.boscavo_volume.Bind(wx.EVT_SLIDER, self.changeBoscavoVolume)
        
        # The Hats Slider
        self.hatsText = wx.StaticText(self.panel, id=-1, label="Hats", pos=(140, 300), size=wx.DefaultSize)
        self.hats = wx.Slider(self.panel, id=-1, value=10, minValue=10, maxValue=100, pos=(140, 320), size=(250, -1))
        self.hats.Bind(wx.EVT_SLIDER, self.changeHats)
        
        # The Accents Slider
        self.accentsText = wx.StaticText(self.panel, id=-1, label="Accents", pos=(140, 430), size=wx.DefaultSize)
        self.accents = wx.Slider(self.panel, id=-1, value=30, minValue=10, maxValue=500, pos=(140, 450), size=(250, -1))
        self.accents.Bind(wx.EVT_SLIDER, self.changeAccents)
        
        # The Boscavo Time Slider
        self.boscavoTimeText = wx.StaticText(self.panel, id=-1, label="Time", pos=(435, 385), size=wx.DefaultSize)
        self.boscavoTime = wx.Slider(self.panel, id=-1, value=25, minValue=1, maxValue=100, pos=(415, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.boscavoTime.Bind(wx.EVT_SLIDER, self.changeBoscavoTime)
        
        # The Onsets Choice 
        self.onsetsChoice = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]
        self.onsets = wx.Choice(self.panel, id=-1, pos=(150, 375), size=wx.DefaultSize, choices=self.onsetsChoice)
        self.onsets.Bind(wx.EVT_CHOICE, self.changeOnsets)
        self.doublePoint = wx.StaticText(self.panel, id=-1, label=":", pos=(255, 375), size=wx.DefaultSize)
        
        # The Taps Choice
        self.tapsChoice = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]
        self.taps = wx.Choice(self.panel, id=-1, pos=(300, 375), size=wx.DefaultSize, choices=self.tapsChoice)
        self.taps.Bind(wx.EVT_CHOICE, self.changeTaps)
        
        
        

### ELARC --------------

        # The Volume Slider
        self.elarc_volume = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=90, pos=(1230, 82), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.elarc_volume.Bind(wx.EVT_SLIDER, self.changeElarcVolume)
        
        # The Harms Slider 
        self.harmsText = wx.StaticText(self.panel, id=-1, label="Harms", pos=(780, 62), size=wx.DefaultSize)
        self.harms = wx.Slider(self.panel, id=-1, value=40, minValue=0, maxValue=100, pos=(780, 82), size=(250, -1))
        self.harms.Bind(wx.EVT_SLIDER, self.changeHarms)
        
        # The Freq Slider
        self.freqText = wx.StaticText(self.panel, id=-1, label="Freq", pos=(780, 192), size=wx.DefaultSize)
        self.freq = wx.Slider(self.panel, id=-1, value=100, minValue=20, maxValue=100, pos=(780, 212), size=(250, -1))
        self.freq.Bind(wx.EVT_SLIDER, self.changeFreq)
        
        # The Low Flow Button
        self.lowFlow = wx.Button(self.panel, id=-1, label="LOW", pos=(790, 137), size=wx.DefaultSize)
        self.lowFlow.Bind(wx.EVT_BUTTON, self.lowFlowButton)
        
        # The Fast Flow Button
        self.fastFlow = wx.Button(self.panel, id=-1, label="FAST", pos=(940, 137), size=wx.DefaultSize)
        self.fastFlow.Bind(wx.EVT_BUTTON, self.fastFlowButton)
        
        # The Feed Slider
        self.feedText = wx.StaticText(self.panel, id=-1, label="Feed", pos=(1070, 145), size=wx.DefaultSize)
        self.feed = wx.Slider(self.panel, id=-1, value=90, minValue=0, maxValue=100, pos=(1050, 82), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.feed.Bind(wx.EVT_SLIDER, self.changeFeed)
        

### SLIFT --------------

        # The Volume Slider
        self.slift_volume = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=90, pos=(1230, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.slift_volume.Bind(wx.EVT_SLIDER, self.changeSliftVolume)
        
        # The White Slider 
        self.whiteText = wx.StaticText(self.panel, id=-1, label="White", pos=(780, 300), size=wx.DefaultSize)
        self.white = wx.Slider(self.panel, id=-1, value=100, minValue=0, maxValue=100, pos=(780, 320), size=(250, -1))
        self.white.Bind(wx.EVT_SLIDER, self.changeWhite)
        
        # The Tone Slider
        self.toneText = wx.StaticText(self.panel, id=-1, label="Tone", pos=(780, 365), size=wx.DefaultSize)
        self.tone = wx.Slider(self.panel, id=-1, value=500, minValue=100, maxValue=1000, pos=(780, 385), size=(250, -1))
        self.tone.Bind(wx.EVT_SLIDER, self.changeTone)
        
        # The Delay Slider
        self.delayText = wx.StaticText(self.panel, id=-1, label="Delay", pos=(780, 430), size=wx.DefaultSize)
        self.delay = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(780, 450), size=(250, -1))
        self.delay.Bind(wx.EVT_SLIDER, self.changeDelay)
        
        # The Time Slider
        self.slifTimeText = wx.StaticText(self.panel, id=-1, label="Time", pos=(1040, 300), size=wx.DefaultSize)
        self.slifTime = wx.Slider(self.panel, id=-1, value=25, minValue=1, maxValue=100, pos=(1050, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.slifTime.Bind(wx.EVT_SLIDER, self.changeSliftTime)

        # The Dur Slider
        self.durText = wx.StaticText(self.panel, id=-1, label="Dur", pos=(1095, 300), size=wx.DefaultSize)
        self.dur = wx.Slider(self.panel, id=-1, value=10, minValue=1, maxValue=100, pos=(1100, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.dur.Bind(wx.EVT_SLIDER, self.changeDur)
        
        # The Cutoff Slider
        self.cutoffText = wx.StaticText(self.panel, id=-1, label="Cutoff", pos=(1135, 300), size=wx.DefaultSize)
        self.cutoff = wx.Slider(self.panel, id=-1, value=10000, minValue=100, maxValue=20000, pos=(1150, 322), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.cutoff.Bind(wx.EVT_SLIDER, self.changeCutoff)
        

### RADZONE --------------

    ### SENDS --------------
        
        # The Send 1 Slider
        self.send1Text = wx.StaticText(self.panel, id=-1, label="CHA", pos=(150, 540), size=wx.DefaultSize)
        self.send1 = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(150, 562), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.send1.Bind(wx.EVT_SLIDER, self.changeChaosSend)
        
        # The Send 2 Slider
        self.send1Text = wx.StaticText(self.panel, id=-1, label="BOS", pos=(200, 540), size=wx.DefaultSize)
        self.send2 = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(200, 562), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.send2.Bind(wx.EVT_SLIDER, self.changeBoscavoSend)
        
        # The Send 3 Slider
        self.send1Text = wx.StaticText(self.panel, id=-1, label="ELA", pos=(250, 540), size=wx.DefaultSize)
        self.send3 = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(250, 562), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.send3.Bind(wx.EVT_SLIDER, self.changeElarcSend)
        
        # The Send 4 Slider
        self.send1Text = wx.StaticText(self.panel, id=-1, label="SLI", pos=(300, 540), size=wx.DefaultSize)
        self.send4 = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(300, 562), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.send4.Bind(wx.EVT_SLIDER, self.changeSliftSend)
        
        # The CHA Madness Button
        self.chaMadness = wx.Button(self.panel, id=-1, label="!", pos=(145, 720), size=(20, 20))
        self.chaMadness.Bind(wx.EVT_BUTTON, self.doChaMadness)
        
        # The BOS Madness Button
        self.bosMadness = wx.Button(self.panel, id=-1, label="!", pos=(195, 720), size=(20, 20))
        self.bosMadness.Bind(wx.EVT_BUTTON, self.doBosMadness)
        
        # The ELA Madness Button
        self.elaMadness = wx.Button(self.panel, id=-1, label="!", pos=(245, 720), size=(20, 20))
        self.elaMadness.Bind(wx.EVT_BUTTON, self.doElaMadness)
        
        # The SLI Madness Button
        self.sliMadness = wx.Button(self.panel, id=-1, label="!", pos=(295, 720), size=(20, 20))
        self.sliMadness.Bind(wx.EVT_BUTTON, self.doSliMadness)
        

    ### ZONE 1 --------------

        # The Zone 1 Slider
        self.zone1Text = wx.StaticText(self.panel, id=-1, label="Zone 1", pos=(370, 542), size=wx.DefaultSize)
        self.zone1 = wx.Slider(self.panel, id=-1, value=50, minValue=0, maxValue=100, pos=(370, 562), size=(250, -1))
        self.zone1.Bind(wx.EVT_SLIDER, self.changeZone1)

        # The Decay Slider
        self.decayText = wx.StaticText(self.panel, id=-1, label="Decay", pos=(370, 607), size=wx.DefaultSize)
        self.decay = wx.Slider(self.panel, id=-1, value=3, minValue=1, maxValue=20, pos=(370, 627), size=(250, -1))
        self.decay.Bind(wx.EVT_SLIDER, self.changeDecay)
        
        # The CHA 1 Toggle
        self.CHAToggle1 = wx.ToggleButton(self.panel, id=-1, label="CHA", pos=(370, 720), size=(50, 20))
        self.CHAToggle1.Bind(wx.EVT_TOGGLEBUTTON, self.CHASend1)
        
        # The BOS 1 Toggle
        self.BOSToggle1 = wx.ToggleButton(self.panel, id=-1, label="BOS", pos=(430, 720), size=(50, 20))
        self.BOSToggle1.Bind(wx.EVT_TOGGLEBUTTON, self.BOSSend1)
        
        # The ELA 1 Toggle
        self.ELAToggle1 = wx.ToggleButton(self.panel, id=-1, label="ELA", pos=(490, 720), size=(50, 20))
        self.ELAToggle1.Bind(wx.EVT_TOGGLEBUTTON, self.ELASend1)
        
        # The SLI 1 Toggle
        self.SLIToggle1 = wx.ToggleButton(self.panel, id=-1, label="SLI", pos=(550, 720), size=(50, 20))
        self.SLIToggle1.Bind(wx.EVT_TOGGLEBUTTON, self.SLISend1)


    ### ZONE 2 --------------
  
        # The Zone 2 Slider
        self.zone2Text = wx.StaticText(self.panel, id=-1, label="Zone 2", pos=(640, 542), size=wx.DefaultSize)
        self.zone2 = wx.Slider(self.panel, id=-1, value=50, minValue=0, maxValue=100, pos=(640, 562), size=(250, -1))
        self.zone2.Bind(wx.EVT_SLIDER, self.changeZone2)

        # The Plate Slider
        self.plateText = wx.StaticText(self.panel, id=-1, label="Plate", pos=(640, 607), size=wx.DefaultSize)
        self.plate = wx.Slider(self.panel, id=-1, value=100, minValue=10, maxValue=1000, pos=(640, 627), size=(250, -1))
        self.plate.Bind(wx.EVT_SLIDER, self.changePlate)
        
        # The Sweep Slider
        #self.sweepText = wx.StaticText(self.panel, id=-1, label="Sweep", pos=(640, 672), size=wx.DefaultSize)
        #self.sweep = wx.Slider(self.panel, id=-1, value=0, minValue=0, maxValue=100, pos=(640, 692), size=(250, -1))
        #self.sweep.Bind(wx.EVT_SLIDER, self.changeSweep)
        
        # The CHA 2 Toggle
        self.CHAToggle2 = wx.ToggleButton(self.panel, id=-1, label="CHA", pos=(640, 720), size=(50, 20))
        self.CHAToggle2.Bind(wx.EVT_TOGGLEBUTTON, self.CHASend2)
        
        # The BOS 2 Toggle
        self.BOSToggle2 = wx.ToggleButton(self.panel, id=-1, label="BOS", pos=(700, 720), size=(50, 20))
        self.BOSToggle2.Bind(wx.EVT_TOGGLEBUTTON, self.BOSSend2)
        
        # The ELA 2 Toggle
        self.ELAToggle2 = wx.ToggleButton(self.panel, id=-1, label="ELA", pos=(760, 720), size=(50, 20))
        self.ELAToggle2.Bind(wx.EVT_TOGGLEBUTTON, self.ELASend2)
        
        # The SLI 2 Toggle
        self.SLIToggle2 = wx.ToggleButton(self.panel, id=-1, label="SLI", pos=(820, 720), size=(50, 20))
        self.SLIToggle2.Bind(wx.EVT_TOGGLEBUTTON, self.SLISend2)
        

    ### ZONE 3 --------------

        # The Zone 3 Slider
        self.zone3Text = wx.StaticText(self.panel, id=-1, label="Zone 3", pos=(915, 542), size=wx.DefaultSize)
        self.zone3 = wx.Slider(self.panel, id=-1, value=50, minValue=0, maxValue=100, pos=(915, 562), size=(250, -1))
        self.zone3.Bind(wx.EVT_SLIDER, self.changeZone3)
        
        # The CHA 3 Toggle
        self.CHAToggle3 = wx.ToggleButton(self.panel, id=-1, label="CHA", pos=(915, 720), size=(50, 20))
        self.CHAToggle3.Bind(wx.EVT_TOGGLEBUTTON, self.CHASend3)
        
        # The BOS 3 Toggle
        self.BOSToggle3 = wx.ToggleButton(self.panel, id=-1, label="BOS", pos=(975, 720), size=(50, 20))
        self.BOSToggle3.Bind(wx.EVT_TOGGLEBUTTON, self.BOSSend3)
        
        # The ELA 3 Toggle
        self.ELAToggle3 = wx.ToggleButton(self.panel, id=-1, label="ELA", pos=(1035, 720), size=(50, 20))
        self.ELAToggle3.Bind(wx.EVT_TOGGLEBUTTON, self.ELASend3)
        
        # The SLI 3 Toggle
        self.SLIToggle3 = wx.ToggleButton(self.panel, id=-1, label="SLI", pos=(1095, 720), size=(50, 20))
        self.SLIToggle3.Bind(wx.EVT_TOGGLEBUTTON, self.SLISend3)
        

### CLOCK --------------

        # The Master Clock Slider
        self.clock = wx.Slider(self.panel, id=-1, value=25, minValue=1, maxValue=100, pos=(650, 320), size=(-1,150), style=wx.SL_VERTICAL | wx.SL_INVERSE)
        self.clock.Bind(wx.EVT_SLIDER, self.changeClock)
        
        # The Sync Button
        self.sync = wx.ToggleButton(self.panel, id=-1, label="SYNC", pos=(615, 480), size=wx.DefaultSize)
        self.sync.Bind(wx.EVT_TOGGLEBUTTON, self.SYNC)
        


### LINES & TEXT --------------
        
        # The Chaos Box
        self.chaosBoxText = wx.StaticText(self.panel, id=-1, label="CHAOS", pos=(494, 38), size=wx.DefaultSize)
        self.chaosBoxText.SetForegroundColour((0, 102, 204))
        self.chaosBoxLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(120, 40), size=(430, 210))
        self.chaosVolumeLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(55, 60), size=(30, 190))
        
        # The Boscavo Box
        self.boscavoBoxText = wx.StaticText(self.panel, id=-1, label="BOSCAVO", pos=(478, 278), size=wx.DefaultSize)
        self.boscavoBoxText.SetForegroundColour((0, 102, 204))
        self.boscavoBoxLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(120, 280), size=(430, 210))
        self.boscavoVolumeLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(55, 300), size=(30, 190))
        
        # The Elarc Box
        self.elarcBoxText = wx.StaticText(self.panel, id=-1, label="ELARC", pos=(765, 38), size=wx.DefaultSize)
        self.elarcBoxText.SetForegroundColour((0, 102, 204))
        self.elarcBoxLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(760, 40), size=(430, 210))
        self.elarcVolumeLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(1225, 60), size=(30, 190))
        
        # The Slift Box
        self.sliftBoxText = wx.StaticText(self.panel, id=-1, label="SLIFT", pos=(765, 278), size=wx.DefaultSize)
        self.sliftBoxText.SetForegroundColour((0, 102, 204))
        self.sliftBoxLine = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(760, 280), size=(430, 210))
        self.sliftVolumeLine = wx.StaticLine(self.panel, -1, style=wx.LI_VERTICAL, pos=(1225, 300), size=(30, 190))
        
        # The Radzone Boxes
        self.radzoneBoxText = wx.StaticText(self.panel, id=-1, label="RADZONE", pos=(1115, 518), size=wx.DefaultSize)
        self.radzoneBoxText.SetForegroundColour((100, 100, 100))
        self.radzoneBoxLine = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(120, 520), size=(1070, 250))
        self.sendLine = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(120, 520), size=(240, 250))
        self.zone1Line = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(353, 520), size=(285, 250))
        self.zone2Line = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(631, 520), size=(268, 250))
        self.zone3Line = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(892, 520), size=(298, 250))
        
        # The Clock Box
        self.clockBoxText = wx.StaticText(self.panel, id=-1, label="CLOCK", pos=(640, 278), size=wx.DefaultSize)
        self.clockBoxText.SetForegroundColour((255, 0, 0))
        self.leftClockText = wx.StaticText(self.panel, id=-1, label="0.250", pos=(610, 390), size=wx.DefaultSize)
        self.leftClockText.SetForegroundColour((255, 0, 0))
        self.rightClockText = wx.StaticText(self.panel, id=-1, label="0.250", pos=(670, 390), size=wx.DefaultSize)
        self.rightClockText.SetForegroundColour((255, 0, 0))
        self.clockBoxLine = wx.StaticLine(self.panel, id=-1, style=wx.LI_VERTICAL, pos=(600, 280), size=(120, 250))


        


#------ utility

            

#------ chaos
        
    def changeChaosVolume(self, evt):
        x = evt.GetInt() * 0.01
        cha.setChaosVolume(x)
            
    def changePitch(self, evt):
        x = evt.GetInt() * 0.01
        cha.setPitch(x)
        chaSend1.setPitch(x)
        chaSend2.setPitch(x)
        chaSend3.setPitch(x)    

    def changeDetune(self, evt):
        x = evt.GetInt() * 0.001
        cha.setDetune(x)
        chaSend1.setDetune(x)
        chaSend2.setDetune(x)
        chaSend3.setDetune(x)

    def changeBalance(self, evt):
        x = evt.GetInt() * 0.01
        cha.setBalance(x)
        chaSend1.setBalance(x)
        chaSend2.setBalance(x)
        chaSend3.setBalance(x)
        
    def changeMovement(self, evt):
        x = evt.GetInt() * 0.01
        cha.setMovement(x)
        chaSend1.setMovement(x)
        chaSend2.setMovement(x)
        chaSend3.setMovement(x)
        
    def toggleChaosMadness(self, evt):
        print "prout"
        
            
 
#------ boscavo
    
    def changeBoscavoVolume(self, evt):
        x = evt.GetInt() * 0.01
        bos.setBoscavoVolume(x)
        
    def changeHats(self, evt):
        x = evt.GetInt() * 0.01
        bos.setHats(x)
        bosSend1.setHats(x)
        bosSend2.setHats(x)
        bosSend3.setHats(x)
        
    def changeAccents(self, evt):
        x = evt.GetInt() * 0.1
        bos.setAccents(x)
        bosSend1.setAccents(x)
        bosSend2.setAccents(x)
        bosSend3.setAccents(x)
        
    def changeOnsets(self, evt):
        x = evt.GetInt()
        for i in range(10):
            if x == i:
                bos.setOnsets(i+1)
                bosSend1.setOnsets(i+1)
                bosSend2.setOnsets(i+1)
                bosSend3.setOnsets(i+1)
                
    def changeTaps(self, evt):
        x = evt.GetInt()
        for i in range(10):
            if x == i:
                bos.setTaps(i+1)
                bosSend1.setTaps(i+1)
                bosSend2.setTaps(i+1)
                bosSend3.setTaps(i+1)
                 
            
    
        
    def changeBoscavoTime(self, evt):
        x = evt.GetInt() * 0.01
        bos.setTime(x)
        bosSend1.setTime(x)
        bosSend2.setTime(x)
        bosSend3.setTime(x)
        self.leftClockText.SetLabel("%.3f" % x)
        self.leftClockText.SetForegroundColour((0, 0, 0))
        

#------ elarc

    def changeElarcVolume(self, evt):
        x = evt.GetInt() * 0.01
        ela.setElarcVolume(x)
        
    def changeHarms(self, evt):
        x = evt.GetInt()
        ela.setHarms(x)
        elaSend1.setHarms(x)
        elaSend2.setHarms(x)
        elaSend3.setHarms(x)
        
    def changeFreq(self, evt):
        x = evt.GetInt() 
        ela.setFreq(x)
        elaSend1.setFreq(x)
        elaSend2.setFreq(x)
        elaSend3.setFreq(x)
        
    def changeFeed(self, evt):
        x = evt.GetInt() * 0.01
        ela.setFeed(x)
        elaSend1.setFeed(x)
        elaSend2.setFeed(x)
        elaSend3.setFeed(x)
        
    def lowFlowButton(self, evt):
        ela.lowFlow()
        elaSend1.lowFlow()
        elaSend2.lowFlow()
        elaSend3.lowFlow()
            
    def fastFlowButton(self, evt):
        ela.fastFlow()
        elaSend1.fastFlow()
        elaSend2.fastFlow()
        elaSend3.fastFlow()
            

#------ slift

    def changeSliftVolume(self, evt):
        x = evt.GetInt() * 0.01
        sli.setSliftVolume(x)
        
    def changeWhite(self, evt):
        x = evt.GetInt() * 0.001
        sli.setWhite(x)
        sliSend1.setWhite(x)
        sliSend2.setWhite(x)
        sliSend3.setWhite(x)
        
    def changeTone(self, evt):
        x = evt.GetInt() 
        sli.setTone(x)
        sliSend1.setTone(x)
        sliSend2.setTone(x)
        sliSend3.setTone(x)
        
    def changeDelay(selg, evt):
        x = evt.GetInt() * 0.01
        sli.setDelay(x)
        sliSend1.setDelay(x)
        sliSend2.setDelay(x)
        sliSend3.setDelay(x)
        
    def changeSliftTime(self, evt):
        x = evt.GetInt() * 0.01
        sli.setTime(x)
        sliSend1.setTime(x)
        sliSend2.setTime(x)
        sliSend3.setTime(x)
        self.rightClockText.SetLabel("%.3f" % x)
        self.rightClockText.SetForegroundColour((0, 0, 0))
        
    def changeDur(self, evt):
        x = evt.GetInt() * 0.001
        sli.setDur(x)
        sliSend1.setDur(x)
        sliSend2.setDur(x)
        sliSend3.setDur(x)
        
    def changeCutoff(self, evt):
        x = evt.GetInt()
        sli.setCutoff(x)
        sliSend1.setCutoff(x)
        sliSend2.setCutoff(x)
        sliSend3.setCutoff(x)
        

#------ Sends Zone
        
    def changeChaosSend(self, evt):
        x = evt.GetInt() * 0.01
        chaSend1.setChaosVolume(x)
        chaSend2.setChaosVolume(x)
        chaSend3.setChaosVolume(x)
        
    def changeBoscavoSend(self, evt):
        x = evt.GetInt() * 0.01
        bosSend1.setBoscavoVolume(x)
        bosSend2.setBoscavoVolume(x)
        bosSend3.setBoscavoVolume(x)

    def changeElarcSend(self, evt):
        x = evt.GetInt() * 0.01
        elaSend1.setElarcVolume(x)
        elaSend2.setElarcVolume(x)
        elaSend3.setElarcVolume(x)
        
    def changeSliftSend(self, evt):
        x = evt.GetInt() * 0.01
        sliSend1.setSliftVolume(x)        
        sliSend2.setSliftVolume(x) 
        sliSend3.setSliftVolume(x) 
        
    def doChaMadness(self, evt):
        print "cha madness"
            
    def doBosMadness(self, evt):
        print "bos madness"
            
    def doElaMadness(self, evt):
        print "ela madness"
            
    def doSliMadness(self, evt):
        print "sli madness"

#------ Zone 1

    def changeZone1(self, evt):
        x = evt.GetInt() * 0.01
        zo1.setZone1(x)
        
    def changeDecay(self, evt):
        x = evt.GetInt()
        zo1.setDecay(x)
        
    def CHASend1(self, evt):
        if(evt.GetInt() == 1):
            chaSend1.setChaosSend(1)
        else:
            chaSend1.setChaosSend(0)
            
    def BOSSend1(self, evt):
        if(evt.GetInt() == 1):
            bosSend1.setBoscavoSend(1)
        else:
            bosSend1.setBoscavoSend(0)
            
    def ELASend1(self, evt):
        if(evt.GetInt() == 1):
            elaSend1.setElarcSend(1)
        else:
            elaSend1.setElarcSend(0)
    
    def SLISend1(self, evt):
        if(evt.GetInt() == 1):
            sliSend1.setSliftSend(1)
        else:
            sliSend1.setSliftSend(0)

#------ Zone 2

    def changeZone2(self, evt):
        x = evt.GetInt() * 0.01
        zo2.setZone2(x)
        
    def changePlate(self, evt):
        x = evt.GetInt() 
        zo2.setPlate(x)
        
    def changeSweep(self, evt):
        x = evt.GetInt() * 0.01
        zo2.setSweep(x)
        
    def CHASend2(self, evt):
        if(evt.GetInt() == 1):
            chaSend2.setChaosSend(1)
        else:
            chaSend2.setChaosSend(0)
            
    def BOSSend2(self, evt):
        if(evt.GetInt() == 1):
            bosSend2.setBoscavoSend(1)
        else:
            bosSend2.setBoscavoSend(0)
            
    def ELASend2(self, evt):
        if(evt.GetInt() == 1):
            elaSend2.setElarcSend(1)
        else:
            elaSend2.setElarcSend(0)
    
    def SLISend2(self, evt):
        if(evt.GetInt() == 1):
            sliSend2.setSliftSend(1)
        else:
            sliSend2.setSliftSend(0)
            

#------ Zone 3

    def changeZone3(self, evt):
        x = evt.GetInt() * 0.01
        zo3.setZone3(x)
        
    def CHASend3(self, evt):
        if(evt.GetInt() == 1):
            chaSend3.setChaosSend(1)
        else:
            chaSend3.setChaosSend(0)
            
    def BOSSend3(self, evt):
        if(evt.GetInt() == 1):
            bosSend3.setBoscavoSend(1)
        else:
            bosSend3.setBoscavoSend(0)
            
    def ELASend3(self, evt):
        if(evt.GetInt() == 1):
            elaSend3.setElarcSend(1)
        else:
            elaSend3.setElarcSend(0)
    
    def SLISend3(self, evt):
        if(evt.GetInt() == 1):
            sliSend3.setSliftSend(1)
        else:
            sliSend3.setSliftSend(0)
            

#------ Clock
    def changeClock(self, evt):
        x = evt.GetInt() * 0.01
        setClock(x)
        self.rightClockText.SetLabel("%.3f" % x)
        self.rightClockText.SetForegroundColour((255, 0, 0))
        self.leftClockText.SetLabel("%.3f" % x)
        self.leftClockText.SetForegroundColour((255, 0, 0))
        
    def SYNC(self, evt):
        if evt.GetInt() == 1:
            bos.stop()
            bosSend1.stop()
            bosSend2.stop()
            bosSend3.stop()
            sli.stop()
            sliSend1.stop()
            sliSend2.stop()
            sliSend3.stop()
        else:
            bos.play()
            bosSend1.play()
            bosSend2.play()
            bosSend3.play()
            sli.play()
            sliSend1.play()
            sliSend2.play()
            sliSend3.play()
        
        
        
app = wx.App()

mainFrame = MyFrame(None, title=NAME + " " + VERSION, pos=(100, 100), size=(1300, 800))
mainFrame.Show()

app.MainLoop()


