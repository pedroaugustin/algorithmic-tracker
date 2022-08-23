<template>
    <div class="home">
        <p class="text-xl" @click="playSection()">Play</p>
        <p class="text-xl" @click="stopPlaying()">Stop</p>
        <p class="text-xl" @click="changeNote()">Change Note</p>
        <select @change="changeTempo" v-model="tempo">
            <option value="80">80</option>
            <option value="120" selected>120</option>
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
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import * as Tone from "tone";
import { Sequence } from "tone";

const synth = new Tone.MonoSynth({
    oscillator: {
        type: "square",
    },
    envelope: {
        attack: 0.01,
        decay: 0.2,
        release: 0.1,
        sustain: 0.1,
    },
    volume: -8,
}).toDestination();

const bassSynth = new Tone.MonoSynth({
    oscillator: {
        type: "square",
    },
    envelope: {
        attack: 0.01,
        decay: 0.2,
        release: 0.1,
        sustain: 0.1,
    },
    volume: -8,
}).toDestination();

const scaleDegreeToHalfSteps = {
    b1: -1,
    1: 0,
    "#1": 1,
    b2: 1,
    sus2: 2,
    2: 2,
    "#2": 3,
    b3: 3,
    3: 4,
    "#3": 5,
    b4: 4,
    sus: 5,
    4: 5,
    "#4": 6,
    b5: 6,
    5: 7,
    "#5": 8,
    b6: 8,
    6: 9,
    "#6": 10,
    d7: 9,
    b7: 10,
    7: 11,
    "#7": 12,
    b8: 11,
    8: 12,
    "#8": 13,
    b9: 13,
    9: 14,
    "#9": 15,
    b10: 15,
    10: 16,
    "#10": 17,
    b11: 16,
    11: 17,
    "#11": 18,
    b12: 18,
    12: 19,
    "#12": 20,
    b13: 20,
    13: 21,
    b14: 22,
    14: 23,
    15: 24,
};

// Scales
const SCALES = {
    MAJOR_SCALE: [0, 2, 4, 5, 7, 9, 11, 12],
    NATURAL_MINOR_SCALE: [0, 2, 3, 5, 7, 8, 10, 12],
};

// Chords
const CHORDS = {
    MAJOR_TRIAD: [0, 4, 7],
    MINOR_TRIAD: [0, 3, 7],
    DIMINISHED_TRIAD: [0, 3, 6],
};

// Patterns
const PATTERNS = {
    II_V_I_MAJOR: [2, 4, 6, 8, 7, 5, 4, 2, 1, 3, 5, 7, 5, 3, 1],
};

// Rythms
const RYTHMS = {
    II_V_I: ["8n", "8n", "8n", "8n", "8n", "8n", "8n", "8n", "8n", "8n", "8n", "4n", "8n", "8n", "8n"],
};

var noteIndex = -1;
var notes = ["C4", "E4", "G4", "B4"];
var durationIndex = -1;
var durations = ["8n", "16n", "16n", "8n"];
var liveNotes = [];
var rythm = [];

function scheduleNext(time) {
    notes = liveNotes
    noteIndex = noteIndex < notes.length - 1 ? noteIndex + 1 : 0;
    var myNote = notes[noteIndex];
    durations = rythm;
    durationIndex = durationIndex < durations.length - 1 ? durationIndex + 1 : 0;

    var myDuration = durations[durationIndex];

    //play note
    synth.triggerAttackRelease(myNote, myDuration, time);
    //schedule the next event relative to the current time by prefixing "+"
    Tone.Transport.schedule(scheduleNext, "+" + myDuration);
}

function playLiveUpdate() {
    if (Tone.Transport.state != "started") {
        noteIndex = -1;
        durationIndex = -1;
        [liveNotes, rythm] = makeMelody(46)
        console.log(notes, rythm)
        scheduleNext("+0.1", notes, rythm);
        Tone.Transport.start("+0.2");
    }
}

function midiToFrequency(midiArray, root) {
    let scale = [];

    for (var i = 0; i < midiArray.length; i++) {
        scale.push(Tone.Frequency(root + midiArray[i], "midi").toFrequency());
    }

    return scale;
}

function chordFromScale(chordArray, scaleArray) {
    let chord = [];

    for (var i = 0; i < chordArray.length; i++) {
        chord.push(scaleArray[chordArray[i]]);
    }

    return chord;
}

function getRandomArbitrary(min, max) {
    return Math.floor(Math.random() * (max - min) + min);
}

function randomIndex(obj) {
    const keys = Object.keys(obj);

    return keys[Math.floor(Math.random() * keys.length)];
}

function patternToSteps(pattern) {
    let steps = [];

    for (var i = 0; i < pattern.length; i++) {
        steps.push(scaleDegreeToHalfSteps[pattern[i]]);
    }

    return steps;
}

function makeMelody(key) {
    let pattern = patternToSteps(PATTERNS['II_V_I_MAJOR']);
    let notes = midiToFrequency(pattern, key);
    let rythm = RYTHMS['II_V_I'];

    // var result = {};
    // for (let i = 0; i < notes.length; i++) {
    //     result[notes[i]] = rythm[i];
    // }

    return [notes, rythm];
}

export default {
    name: "HomeView",
    components: {
        HelloWorld,
        Tone,
    },
    data() {
        return {
            tempo: 120,
            scales: null,
            chords: null,
        };
    },
    created() {},
    methods: {
        changeTempo() {
            Tone.Transport.bpm.value = this.tempo;
        },
        playSection() {
            // Tone.start();
            // Tone.Transport.bpm.value = this.tempo;
            // Tone.Transport.start();
            playLiveUpdate();
        },
        stopPlaying() {
            Tone.Transport.stop();
            Tone.Transport.cancel(0);
        },
        changeNote() {},
    },
};
</script>
