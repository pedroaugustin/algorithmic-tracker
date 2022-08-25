<template>
    <div class="home">
        <p class="text-xl" @click="startPlaying()">Play</p>
        <p class="text-xl" @click="stopPlaying()">Stop</p>
        <p class="text-xl" @click="changeNote()">Change Note</p>
        <select @change="changeTempo" v-model="tempo">
            <option value="80">80</option>
            <option value="120">120</option>
            <option value="140">140</option>
        </select>
        <div>
            <p v-for="scale in scales" :key="scale" class="inline m-2">{{ scale }}</p>
        </div>
        <div>
            <p v-for="chord in chords" :key="chord" class="inline m-2">{{ chord }}</p>
        </div>
    </div>
</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import { sequenceExpression } from "@babel/types";
import { NodeTypes } from "@vue/compiler-core";
import * as Tone from "tone";
import { Sequence } from "tone";
import * as Serialism from "total-serialism";

const synth = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "square",
    },
    envelope: {
        attack: 0.05,
        attackCurve: 'linear',
        decay: 0.3,
        decayCurve: 'exponential',
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.4
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: 'lowpass'
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: 'linear',
        baseFrequency: 300,
        decay: 0.7,
        decayCurve: 'exponential',
        exponent: 2,
        octaves: 4,
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.1
    },
    volume: -16,
}).toDestination();

const synth2 = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "square",
    },
    envelope: {
        attack: 0.05,
        attackCurve: 'linear',
        decay: 0.3,
        decayCurve: 'exponential',
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.4
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: 'lowpass'
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: 'linear',
        baseFrequency: 300,
        decay: 0.7,
        decayCurve: 'exponential',
        exponent: 2,
        octaves: 4,
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.1
    },
}).toDestination();

const bassSynth = new Tone.MonoSynth({
    oscillator: {
        frequency: 440,
        type: "square",
    },
    envelope: {
        attack: 0.05,
        attackCurve: 'linear',
        decay: 0.3,
        decayCurve: 'exponential',
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.4
    },
    filter: {
        Q: 1,
        rollof: -12,
        type: 'lowpass'
    },
    filterEnvelope: {
        attack: 0.001,
        attackCurve: 'linear',
        baseFrequency: 300,
        decay: 0.7,
        decayCurve: 'exponential',
        exponent: 2,
        octaves: 4,
        release: 0.8,
        releaseCurve: 'exponential',
        sustain: 0.1
    },
    volume: -16,
}).toDestination();

const Algo = Serialism.Algorithmic

// Scales
const SCALES = {
    MAJOR: [0, 2, 4, 5, 7, 9, 11, 12],
    MINOR: [0, 2, 3, 5, 7, 8, 10, 12],
    MELODIC_MINOR: [0, 2, 3, 5, 7, 9, 11, 12],
    DORIAN: [0, 2, 3, 5, 7, 9, 10, 12],
    PHRYGIAN: [0, 1, 3, 5, 7, 8, 10, 12],
    LYDIAN: [0, 2, 4, 6, 7, 9, 11, 12],
    MIXOLYDIAN: [0, 2, 4, 5, 7, 9, 10, 12],
    LOCRIAN: [0, 1, 3, 5, 6, 8, 10, 12],
    NATURAL_MINOR: [0, 2, 3, 5, 7, 8, 10, 12],
    MAJOR_PENTATONIC: [0, 2, 4, 7, 9, 12, 14, 16, 19, 21, 24],
    MINOR_PENTATONIC: [0, 3, 5, 7, 10, 12, 15, 17, 19, 22, 24],

};

// Chords
const CHORDS = {
    MAJOR_TRIAD: [0, 4, 7],
    MINOR_TRIAD: [0, 3, 7],
    DIMINISHED_TRIAD: [0, 3, 6],
};

const PATTERNS = [
    [0, 0, 0, 0, 5, 5, 5, 5, 3, 3, 3, 3, 2, 2, 2, 2]
]

let numSynthNotes = 12;
let numSynth2Notes = 64;
let numBassNotes = 4;
let bassMelody = ['C4'];
let synthMelody = ['C4'];
let synth2Melody = ['C4'];
let root = 52
let synthRoot = root
let bassRoot = synthRoot - 24
let bpm = 120

// Synths Patterns and Sequences
const synthPattern = new Tone.Pattern((time, note) => {
    synth.triggerAttackRelease(note, "32n", time);
}, synthMelody, "random").start(0);

const bassSeq = new Tone.Sequence((time, note) => {
    bassSynth.triggerAttackRelease(note, "16n", time);
}, bassMelody).start(0);

const synth2Part = new Tone.Part(((time, value) => {
    synth2.triggerAttackRelease(value.note, "4n", time, value.velocity);
})).start(0);

synth2Part.loop = true
synth2Part.loopStart = 0
synth2Part.loopEnd = "4m"

function fillMelodyArray(melody) {
    let seconds = 30 / bpm
    let melodyArray = []
    let time = 0

    for (let i = 0; i < 64; i++) {
        if (melody[i] != 0) {
            melodyArray.push({ time: time, note: Tone.Frequency(melody[i], 'midi').toFrequency(), velocity: 0.5 })
        }
        time = time + seconds
    }

    return melodyArray
}

function generateBassMelody() {
    bassMelody = []

    // randomizar esse
    let euclid = Algo.euclid(64, 54)
    let auxIndex = 0
    let bassPattern = 0
    let note

    if (bassPattern == 0) {
        //euclidean
        for (let i = 0; i < PATTERNS[0].length; i++) {
            for (let j = 0; j < numBassNotes; j++) {
                //randomizar pattern
                note = (bassRoot + PATTERNS[0][i]) * euclid[auxIndex];
                if (note == 0) {
                    bassMelody.push(0);
                } else {
                    note = Tone.Frequency(note, 'midi')
                    bassMelody.push(note);
                }

                auxIndex++
            }
        }
    } else {
        //1,2
        for (let i = 0; i < PATTERNS[0].length; i++) {
            for (let j = 0; j < numBassNotes; j++) {
                //randomizar pattern
                if (auxIndex % 2 == 0) {
                    note = (bassRoot + PATTERNS[0][i]);
                } else {
                    note = (bassRoot + PATTERNS[0][i]) + 12
                }
                note = Tone.Frequency(note, 'midi')
                bassMelody.push(note);

                auxIndex++
            }
        }
    }

    console.log(bassMelody);
}

function generateSynth2Melody() {
    synth2Melody = []

    // randomizar esse
    let euclid = Algo.fastEuclid(64, 42)
    let auxIndex = 0
    let synth2Notes = []
    let note

    //euclidean
    for (let i = 0; i < numSynth2Notes; i++) {
        note = (synthRoot + SCALES['MINOR_PENTATONIC'][Math.floor(Math.random() * SCALES['MINOR_PENTATONIC'].length)]) * euclid[auxIndex];
        synth2Notes.push(note);
        auxIndex++
    }

    synth2Melody = fillMelodyArray(synth2Notes);

    for (let i = 0; i < synth2Melody.length; i++) {
        synth2Part.add(synth2Melody[i]);
    }

    console.log(synth2Melody)
    console.log(synth2Notes);
}

function generateSynthMelody() {
    synthMelody = []
    let synthNotes = []

    for (let i = 0; i < numSynthNotes; i++) {
        let note = synthRoot + SCALES['MINOR'][Math.floor(Math.random() * SCALES['MINOR'].length)];
        synthNotes.push(note);
        note = Tone.Frequency(note, 'midi').toFrequency()
        synthMelody.push(note);
    }
    synthMelody.sort()

    console.log(synthNotes);
}

function setMelodies() {
    bassSeq.set({
        events: bassMelody
    })
    synthPattern.set({
        values: synthMelody
    })
    synth2Part.add(synth2Melody)
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
        Tone.Transport.timeSignature = [4, 4]
        Tone.Transport.loop = true
        Tone.Transport.loopStart = 0
        Tone.Transport.loopEnd = "16m"

        synthPattern.interval = "16n"
    },
    methods: {
        changeTempo() {
            Tone.Transport.bpm.rampTo(this.tempo, 0.001);
        },
        startPlaying() {
            Tone.start()
            generateBassMelody();
            generateSynthMelody();
            generateSynth2Melody();

            setMelodies();
            //Tone.Transport.scheduleRepeat(setBassMelody, '8n');
            //Tone.Transport.scheduleRepeat(console.log(Tone.Transport.beat), '4n');
            Tone.Transport.start("+0.1");

        },
        stopPlaying() {
            if (Tone.Transport.state == "started") {
                Tone.Transport.stop();
                Tone.Transport.cancel(0);
            }
        },
        changeNote() {
            synthRoot = synthRoot - 1
            bassRoot = bassRoot - 1
            generateBassMelody();
            generateSynthMelody();
            generateSynth2Melody();
            setMelodies();
        },
    },
};
</script>
