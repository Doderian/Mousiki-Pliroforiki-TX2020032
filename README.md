s.boot;


(
t = TempoClock(120/60).permanent_(true); //den feugei meta apo to ctrl + .



(
b = Buffer.read(s, Platform.resourceDir +/+ "sounds/a11wlk01.wav");
)

// SynthDef for granular synthesis
(
SynthDef(\granulator, {
    arg bufnum, rate = 1, startPos = 0, dur = 4, amp=10, density = 10, pan = 0;
    var env, grains, grain, readPos, playbackRate;
    env = EnvGen.kr(Env.perc, doneAction: 2);
    playbackRate = BufRd.kr(1, bufnum, rate * BufRateScale.kr(bufnum));
    readPos = Phasor.kr(0, BufFrames.kr(bufnum) / BufRateScale.kr(bufnum), 0, dur);
    grain = GrainBuf.ar(2, Impulse.kr(density), dur, bufnum, readPos, 1, 2, 0.1);
    grains = grain * env;
    grains = Pan2.ar(grains, pan,amp);
    Out.ar(0, grains);
}).add;
)



SynthDef.new(\kick,{
	var freqA=250, freqB=20, freqC=10, freqDur1=0.01, freqDur2=0.2, freqC1=1, freqC2=(-1),
	atk=0.01,rel=1,c1=1,c2=(-12),amp=2,pan=0,out=0;
	var sig, env, freqSweep;
	freqSweep = Env([freqA, freqB, freqC], [freqDur1,freqDur2], [freqC1, freqC2]).ar;
	env = Env([0,1,0], [atk,rel], [c1,c2]).kr(2);
	sig = SinOsc.ar(freqSweep, pi/2);
	sig = sig * env;
	sig = Pan2.ar(sig,pan,amp);
	Out.ar(out,sig);
}).add;




SynthDef(\bass, { |out, freq = 440, gate = 1, amp = 0.2, slideTime = 0.17, ffreq = 1100, width = 0.15,
        detune = 1.005, preamp = 4|
    var sig, env;
    env = Env.adsr(0.01, 0.3, 0.4, 0.1);
    freq = Lag.kr(freq, slideTime);
    sig = Mix(VarSaw.ar([freq, freq * detune], 0, width, preamp)).distort * amp
        * EnvGen.kr(env, gate, doneAction: Done.freeSelf);
    sig = LPF.ar(sig, ffreq);
    Out.ar(out, sig ! 2)
}).add;

Synth(\bass);




SynthDef(\bassSynth, {
	|freq = 440, amp = 0.5, gate = 1|
	var env = Env.adsr(0.01,0.2, 0.4,0.5).kr(gate: gate);
	var osc = Saw.ar(freq) * env;
	var sound = LPF.ar(osc, 1000) *amp;
	Out.ar(0,Pan2.ar(sound));
}).add;




x = Synth(\kick);

SynthDef(\snare, {
    var sig, freq;
	var amp = 0.5;
    freq = 230 * (1 + (Env.perc(0.001, 0.03).ar * 2));
    sig = SinOsc.ar(freq + (200 * SinOsc.ar(freq * 1.2))) * Env.perc(0.001, 0.2).ar;
    sig = sig + (BPF.ar(Hasher.ar(Sweep.ar), 1000, 0.3) * Env.perc(0.1, 0.3).ar);
    sig = sig.tanh;
    sig = sig * Env.perc(0.001, 5).ar(Done.freeSelf);
    sig = sig ! 2;
	sig = sig + (FreeVerb.ar(sig,1, 0.5, 0.5));
    sig = sig * \amp.kr(1);
    Out.ar(\out.kr(0), sig);
}).add;

Synth(\snare);


//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(
{
Pbind(
	\instrument, \kick,
		\dur, Pseq ([0.58], inf),
	\amp, 1
).play; //tempo clock



Pbind(
    \instrument, \granulator,
    \bufnum, b.bufnum,
    \rate, Pseq([1, 1.5, 0.5], inf),
    \dur, Pseq([4, 2, 6], inf),
    \density, Pseq([10, 5, 15], inf),
    \pan, Pseq([0.2, -0.2, 0.5, -0.5], inf),
	\amp, 8
).play;

Pbind(
    \instrument, \bassSynth,
		\midinote, Pseq ([
		Pseq([33,45,33,45,33,40,43,45],1) ++
		Pseq([31,43,31,43,33,40,43,45],1) ++
		Pseq([33,45,33,45,43,40,45,40],1) ++
		Pseq([31,43,31,43,43,40,39,38],1),
		],inf),
	\dur, Pseq([0.41,0.25,0.22,0.28,0.29,0.29,0.29,0.29],inf),
    \amp, 1.2
).play;


Pbind(
    \instrument, \bass,
		\midinote, Pwrand([33,45,33,45,33,40,43,45], [0.25,0.1,0.05,0.05,0.125,0.125,0.175,0.125],inf),
	\ctranspose, 24,
		\dur, Prand([0.29,0.145],inf),
    \amp, 0.6,
	\legato, 1.2
).play;

Pbind(
	\instrument, \snare,
	\dur, 1.16,
	\amp, 0.2
	).play;

9.68.wait;

Pbind(
	\instrument, \bass,
	\note, Pseq([[-3,0,4],[-3,0,4],[-5,-1,2],[-5,-1,2]],inf),
	\dur, Pseq([0.26,2.06,0.26,2.06],inf),
	\amp, 0.4,
	\legato, 0.3
).play;
}.fork;
)
