<template>
    <div class="home">
        <p class="text-xl" @click="startPlaying()">Play</p>
        <p class="text-xl" @click="stopPlaying()">Stop</p>
        <p class="text-xl" @click="changeNote()">Change Note</p>
        <select @change="changeTempo" v-model="tempo">
            <option value="80">80</option>
            <option value="120">120</option>
            <option value="130">130</option>
        </select>
        <div>
            <p v-for="scale in scales" :key="scale" class="inline m-2">{{  scale  }}</p>
        </div>
        <div>
            <p v-for="chord in chords" :key="chord" class="inline m-2">{{  chord  }}</p>
        </div>
    </div>
</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import * as Tone from "tone";
import * as Serialism from "total-serialism";

const reverb = new Tone.Reverb().toDestination();
const tremolo = new Tone.Tremolo(8, 1).toDestination().start();

const Rand = Serialism.Stochastic;
Rand.seed(666)

const hihatSynth = new Tone.NoiseSynth({
    envelope: {
        attack: 0.001,
        decay: 0.001,
        sustain: 0.8,
        release: 0.001,
    },
    volume: -20,
}).toDestination();

const kickSynth = new Tone.MembraneSynth({
    pitchDecay: 0.20,
    octaves: 12,
    oscillator: {
        type: "triangle"
    },
    envelope: {
        attack: 0.01,
        decay: 0.001,
        sustain: 1,
        release: 0.001,
    },
    volume: -6
}).toDestination();

const snareSynth = new Tone.NoiseSynth({
    envelope: {
        attack: 0.0001,
        decay: 0.001,
        sustain: 1,
        release: 0.15,
    },
    volume: -24,
}).toDestination();

const synth = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "sawtooth",
    },
    envelope: {
        attack: 0.001,
        attackCurve: "linear",
        decay: 0.001,
        decayCurve: "exponential",
        sustain: 1,
        release: 0.1,
        releaseCurve: "exponential",
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: "lowpass",
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: "linear",
        baseFrequency: 400,
        decay: 0.01,
        decayCurve: "exponential",
        exponent: 2,
        octaves: 4,
        sustain: 1,
        release: 0.8,
        releaseCurve: "exponential",
    },
    volume: -25,
}).toDestination();

const synth2 = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "square",
    },
    envelope: {
        attack: 0.001,
        attackCurve: "linear",
        decay: 0.001,
        decayCurve: "exponential",
        sustain: 1,
        release: 0.1,
        releaseCurve: "exponential",
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: "lowpass",
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: "linear",
        baseFrequency: 400,
        decay: 0.01,
        decayCurve: "exponential",
        exponent: 2,
        octaves: 4,
        sustain: 1,
        release: 0.8,
        releaseCurve: "exponential",
    },
    volume: -24,
}).toDestination().connect(reverb);

const synth3 = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "sawtooth",
    },
    portamento: 0.05,
    envelope: {
        attack: 0.001,
        attackCurve: "linear",
        decay: 0.001,
        decayCurve: "exponential",
        sustain: 1,
        release: 0.8,
        releaseCurve: "exponential",
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: "lowpass",
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: "linear",
        baseFrequency: 400,
        decay: 0.01,
        decayCurve: "exponential",
        exponent: 2,
        octaves: 8,
        sustain: 1,
        release: 0.8,
        releaseCurve: "exponential",
    },
    volume: -30,
}).connect(tremolo).connect(reverb).toDestination();

const bassSynth = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "sawtooth",
    },
    envelope: {
        attack: 0.001,
        attackCurve: "linear",
        decay: 0.001,
        decayCurve: "exponential",
        sustain: 1,
        release: 0.06,
        releaseCurve: "exponential",
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: "lowpass",
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: "linear",
        baseFrequency: 400,
        decay: 0.01,
        decayCurve: "exponential",
        exponent: 2,
        octaves: 4,
        sustain: 1,
        release: 0.8,
        releaseCurve: "exponential",
    },
    volume: -20,
}).toDestination();

const Algo = Serialism.Algorithmic;

// Scales
const SCALES = {
    MAJOR: [0, 2, 4, 5, 7, 9, 11, 12],
    MINOR: [0, 2, 3, 5, 7, 8, 10, 12],
    MELODIC_MINOR: [0, 2, 3, 5, 7, 9, 11, 12],
    DORIAN: [0, 2, 3, 5, 7, 9, 10, 12],
    PHRYGIAN: [0, 1, 3, 5, 7, 8, 10, 12],
    NATURAL_MAJOR: [0, 2, 4, 6, 7, 9, 11, 12],
    MIXOLYDIAN: [0, 2, 4, 5, 7, 9, 10, 12],
    LOCRIAN: [0, 1, 3, 5, 6, 8, 10, 12],
    NATURAL_MINOR: [0, 2, 3, 5, 7, 8, 10, 12],
    MAJOR_PENTATONIC: [0, 2, 4, 7, 9, 12, 14, 16, 19, 21, 24],
    MINOR_PENTATONIC: [0, 3, 5, 7, 10, 12, 15, 17, 19, 22, 24],
};

const PROGRESSIONS = [
    [0, 0, 0, 0, 5, 5, 5, 5, 3, 3, 3, 3, 2, 2, 2, 2], // I-VI-IV-III
    [0, 0, 0, 0, 3, 3, 3, 3, 0, 0, 0, 0, 6, 6, 6, 6],
];

const FIFTHS = [60, 67, 62, 69, 64, 71, 66, 61, 68, 63, 70, 65];

const CHORDS = {
    MAJOR: [0, 4, 7],
    MAJOR_7: [0, 4, 7, 11],
    MINOR: [0, 3, 7],
    MINOR_7: [0, 3, 7, 11],
    DIMINISHED: [0, 3, 6],
}

let numBassNotes = 4;
let bassMelody = ["C4"];
let synthMelody = ["C4"];
let synth2Melody = [];
let synth3Melody = [];
let kickPattern = []
let hihatPattern = []
let snarePattern = []
let fifthIndex = 0;
let root = 0;
let synthRoot = root;
let bassRoot = root - 24;
let bpm = 130;

// Synths Patterns and Sequences
const synthPattern = new Tone.Pattern(
    (time, note) => {
        synth.triggerAttackRelease(note, "32n", time);
    },
    synthMelody,
    "upDown"
).start("+0.1");

const bassSeq = new Tone.Sequence((time, note) => {
    bassSynth.triggerAttackRelease(note, "16n", time);
}, bassMelody).start("+0.1");

const synth2Part = new Tone.Part((time, value) => {
    synth2.triggerAttackRelease(value.note, "32n", time, value.velocity);
})//.start("+0.1");

const synth3Part = new Tone.Part((time, value) => {
    synth3.triggerAttackRelease(value.note, value.duration, time, value.velocity);
}).start("+0.1");

const kickPart = new Tone.Part((time, value) => {
    kickSynth.triggerAttackRelease(value.note, "32n", time, value.velocity);
})//.start("+0.1");

const hhPart = new Tone.Part((time, value) => {
    hihatSynth.triggerAttackRelease("64n", time, value.velocity);
})//.start("+0.1");

const snarePart = new Tone.Part((time, value) => {
    snareSynth.triggerAttackRelease("32n", time, value.velocity);
})//.start("+0.1");


function fillDrums() {
    let seconds = Tone.Time("16n").toSeconds();
    kickPattern = [];
    hihatPattern = [];
    snarePattern = [];
    let time = 0;

    kickPart.clear()
    hhPart.clear()
    snarePart.clear()

    for (let i = 0; i < 256; i++) {
        if (i % 8 == 0 || i == 0) {
            kickPattern.push({ time: time, note: 20, velocity: 1 });
        } else if (i % 8 == 4) {
            snarePattern.push({ time: time, note: 60, velocity: 0.8 });
        } else if (i % 2 == 0) {
            hihatPattern.push({ time: time, note: 50, velocity: 1 });
        } else if (Rand.pick(1, [1, 2, 3, 4, 5]) == 1) {
            snarePattern.push({ time: time, note: 20, velocity: 1 });
        } else {
            hihatPattern.push({ time: time, note: 20, velocity: 1 });
        }

        time = time + seconds;
    }

    for (let i = 0; i < kickPattern.length; i++) {
        kickPart.add(kickPattern[i]);
    }

    for (let i = 0; i < hihatPattern.length; i++) {
        hhPart.add(hihatPattern[i]);
    }

    for (let i = 0; i < snarePattern.length; i++) {
        snarePart.add(snarePattern[i]);
    }

    //console.log(hihatPattern)
    //console.log(kickPattern)
}

function fillMelodyArray(melody) {
    let seconds = Tone.Time("16n").toSeconds();
    let melodyArray = [];
    let time = 0;

    for (let i = 0; i < 128; i++) {
        if (melody[i] != 0) {
            melodyArray.push({ time: time, duration: i % 2 == 0 ? "4n" : "32n", note: Tone.Frequency(melody[i], "midi").toFrequency(), velocity: 1 });
        }
        time = time + seconds;
    }

    return melodyArray;
}

function fillMelody8nArray(melody) {
    let seconds = Tone.Time("8n").toSeconds();
    let melodyArray = [];
    let time = 0;

    for (let i = 0; i < 64; i++) {
        if (melody[i] != 0) {
            melodyArray.push({ time: time, duration: i % 2 == 0 ? "4n" : "32n", note: Tone.Frequency(melody[i], "midi").toFrequency(), velocity: 1 });
        }
        time = time + seconds;
    }

    return melodyArray;
}

function generateBassMelody() {
    bassMelody = [];

    // randomizar esse
    let euclid = Algo.euclid(64, 54);
    let auxIndex = 0;
    let bassPattern = 0;
    let note;

    if (bassPattern == 0) {
        //euclidean
        for (let i = 0; i < PROGRESSIONS[0].length; i++) {
            for (let j = 0; j < numBassNotes; j++) {
                //randomizar pattern
                note = (bassRoot + PROGRESSIONS[0][i]) * euclid[auxIndex];
                if (note == 0) {
                    bassMelody.push(0);
                } else {
                    note = Tone.Frequency(note, "midi").toFrequency();
                    bassMelody.push(note);
                }

                auxIndex++;
            }
        }
    } else {
        //1,2
        for (let i = 0; i < PROGRESSIONS[0].length; i++) {
            for (let j = 0; j < numBassNotes; j++) {
                //randomizar pattern
                if (auxIndex % 2 == 0) {
                    note = bassRoot + PROGRESSIONS[1][i];
                } else {
                    note = bassRoot + PROGRESSIONS[1][i] + 12;
                }
                note = Tone.Frequency(note, "midi").toFrequency();
                bassMelody.push(note);

                auxIndex++;
            }
        }
    }

    console.log(bassMelody);
}

function generateSynthMelody() {
    synthMelody = [];
    let note = 0;
    let octaveNote = 0;

    for (let i = 0; i < SCALES["MINOR"].length; i++) {
        note = synthRoot + SCALES["MINOR"][i];
        octaveNote = note + 12
        note = Tone.Frequency(note, "midi").toFrequency();
        octaveNote = Tone.Frequency(octaveNote, "midi").toFrequency();
        synthMelody.push(note);
        synthMelody.push(octaveNote);
    }

    synthMelody.sort(function (a, b) { return a - b });
    //console.log(synthMelody);
}

function generateSynth2Melody() {
    synth2Melody = [];
    synth2Part.clear();

    // randomizar esse
    let euclid = Algo.fastEuclid(128, 78);
    let auxIndex = 0;
    let synth2Notes = [];
    let note;

    //euclidean
    for (let i = 0; i < PROGRESSIONS[1].length; i++) {
        for (let j = 0; j < 8; j++) {
            note = (synthRoot + PROGRESSIONS[1][i] + (SCALES["MINOR_PENTATONIC"][Math.floor(Math.random() * SCALES["MINOR_PENTATONIC"].length)])) * euclid[auxIndex];
            synth2Notes.push(note);
            auxIndex++;
        }
    }

    synth2Melody = fillMelodyArray(synth2Notes);

    for (let i = 0; i < synth2Melody.length; i++) {
        synth2Part.add(synth2Melody[i]);
    }

    //console.log(synth2Melody);
}

function generateSynth3Melody() {
    synth3Melody = [];
    synth3Part.clear();

    // randomizar esse
    let auxIndex = 0;
    let synth3Notes = [];
    let note;

    let ca = new Algo.Automaton();
    // feed with 40 randomly generated values 0-1
    ca.feed(Rand.coin(128));
    ca.rule(122);
    let gen = ca.next();

    //euclidean
    for (let i = 0; i < PROGRESSIONS[1].length; i++) {
        for (let j = 0; j < 4; j++) {
            note = (synthRoot + PROGRESSIONS[1][i] + (CHORDS["MINOR"][Math.floor(Math.random() * CHORDS["MINOR"].length)])) * gen[auxIndex];
            synth3Notes.push(note);
            auxIndex++;
        }
    }

    console.log(synth3Notes);
    synth3Melody = fillMelody8nArray(synth3Notes);

    for (let i = 0; i < synth3Melody.length; i++) {
        synth3Part.add(synth3Melody[i]);
    }

    //console.log("ae" , synth3Melody);
}

function setMelodies() {
    bassSeq.set({
        events: bassMelody,
    });
    synthPattern.set({
        values: synthMelody,
    });
}

function setup() {
    Tone.Transport.timeSignature = [4, 4];
    Tone.Transport.loop = true;
    Tone.Transport.loopStart = 0;
    Tone.Transport.loopEnd = "8m";
    synthPattern.interval = "16n";
    synth2Part.loop = true;
    synth2Part.loopStart = 0;
    synth2Part.loopEnd = "8m";
    synth3Part.loop = true;
    synth3Part.loopStart = 0;
    synth3Part.loopEnd = "8m";
    kickPart.loop = true;
    kickPart.loopStart = 0;
    kickPart.loopEnd = "8m";
    hhPart.loop = true;
    hhPart.loopStart = 0;
    hhPart.loopEnd = "8m";
    fifthIndex = Math.floor(Math.random() * FIFTHS.length)
    root = FIFTHS[fifthIndex];
}

function setRoot() {
    let upOrDown = Math.floor(Math.random() * 10)

    if (upOrDown % 2 == 0) {
        if (fifthIndex + 1 > FIFTHS.length) {
            fifthIndex = FIFTHS[0];
        } else {
            fifthIndex = fifthIndex + 1;
        }
    } else {
        if (fifthIndex - 1 < 0) {
            fifthIndex = FIFTHS[FIFTHS.length];
        } else {
            fifthIndex = fifthIndex - 1;
        }
    }

    root = FIFTHS[fifthIndex]
    synthRoot = root
    bassRoot = root - 36
}

function generateNewPatterns() {
    setRoot();
    generateBassMelody();
    generateSynthMelody();
    generateSynth2Melody();
    generateSynth3Melody();
    setMelodies();
    fillDrums();
}

export default {
    name: "HomeView",
    components: {
        HelloWorld,
        Tone,
    },
    data() {
        return {
            tempo: bpm,
            scales: null,
            chords: null,
        };
    },
    created() {
        setup();
    },
    methods: {
        changeTempo() {
            Tone.Transport.bpm.rampTo(this.tempo, 0.001);
        },
        startPlaying() {
            Tone.start();
            Tone.Transport.bpm.rampTo(this.tempo, 0.001);
            Tone.Transport.scheduleRepeat(time => {
                generateNewPatterns();
            }, "8m");
            Tone.Transport.start("+0.1");
        },
        stopPlaying() {
            if (Tone.Transport.state == "started") {
                Tone.Transport.stop();
                Tone.Transport.cancel(0);
            }
        },
        changeNote() {
            Tone.start();
            synth3.triggerAttackRelease("C4", "2n");
        },
    },
};
</script>
