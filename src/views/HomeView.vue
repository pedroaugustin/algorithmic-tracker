<template>
    <div class="home">
        <div class="flex justify-center mb-2">
            <input
                class="text-[20px] bg-slate-100 shadow w-[340px] h-12 border-2 p-2 border-black text-black rounded-md font-['Karma']"
                v-model="textSeed"
                type="text"
                placeholder="Type Here"
            />
            <div class="h-12">
                <p
                    class="text-[50px] leading-10 text-white ml-2 font-['8_Bit_Arcade'] playText cursor-pointer hover:text-[#37d706]"
                    @click="startPlaying()"
                    v-if="!playing"
                >
                    play
                </p>
                <p
                    class="text-[50px] leading-10 text-white ml-2 font-['8_Bit_Arcade'] playText cursor-pointer hover:text-[#37d706]"
                    @click="stopPlaying()"
                    v-if="playing && !paused"
                >
                    pause
                </p>
                <p
                    class="text-[50px] leading-10 text-white ml-2 font-['8_Bit_Arcade'] playText cursor-pointer hover:text-[#37d706]"
                    @click="resumePlaying()"
                    v-if="paused"
                >
                    resume
                </p>
            </div>
            <div class="h-12">
                <p
                    class="text-[30px] leading-10 text-white ml-2 font-['8_Bit_Arcade'] playText cursor-pointer hover:text-[#37d706]"
                    @click="resetPlaying()"
                >
                    reset
                </p>
            </div>
        </div>
        <span class="text-[10px] text-gray-900 p-2 font-['8_Bit_Hud']"
            >Type a word or press play to generate a random seed</span
        >
        <div
            class="ml-16 mr-16 mt-2 border-4 bg-slate-50/80 rounded-md border-neutral-800 divide-slate-400 divide-x-2 divide-y-2 shadow grid grid-cols-5"
        >
            <div>
                <p class="menuTitle">Tempo</p>
                <input @change="changeTempo" v-model="tempo" type="range" min="60" max="150" />
                <span> {{ tempo }} BPM</span>
            </div>
            <div class="flex items-center justify-center relative" :class="{ 'bg-gray-400/80': !enable1 }">
                <input v-model="enable1" class="absolute m-1 top-0 left-0" type="checkbox" />
                <div>
                    <p class="menuTitle">Root</p>
                    <select v-model="context.root" :disabled="!enable1" class="m-1">
                        <option value="">Rnd</option>
                        <template v-for="(rootNote, index) in rootNotes" :key="index">
                            <option :value="rootNote">{{ rootNote }}</option>
                        </template>
                    </select>
                </div>
                <div>
                    <p class="menuTitle">Octave</p>
                    <select v-model="context.octave" :disabled="!enable1" class="m-1">
                        <option value="">Rnd</option>
                        <option :value="0">0</option>
                        <option :value="1">1</option>
                        <option :value="2">2</option>
                        <option :value="3">3</option>
                    </select>
                </div>
            </div>
            <div class="relative" :class="{ 'bg-gray-400/80': !enable2 }">
                <input v-model="enable2" class="absolute m-1 top-0 left-0" type="checkbox" />
                <p class="menuTitle">Progression</p>
                <select v-model="context.progression[0]" :disabled="!enable2" class="m-1">
                    <option value="">Rnd</option>
                    <template v-for="(chord, index) in chords" :key="index">
                        <option :value="chord">{{ chord }}</option>
                    </template>
                </select>
                <select v-model="context.progression[1]" :disabled="!enable2" class="m-1">
                    <option value="">Rnd</option>
                    <template v-for="(chord, index) in chords" :key="index">
                        <option :value="chord">{{ chord }}</option>
                    </template>
                </select>
                <select v-model="context.progression[2]" :disabled="!enable2" class="m-1">
                    <option value="">Rnd</option>
                    <template v-for="(chord, index) in chords" :key="index">
                        <option :value="chord">{{ chord }}</option>
                    </template>
                </select>
                <select v-model="context.progression[3]" :disabled="!enable2" class="m-1">
                    <option value="">Rnd</option>
                    <template v-for="(chord, index) in chords" :key="index">
                        <option :value="chord">{{ chord }}</option>
                    </template>
                </select>
            </div>
        </div>
        <div
            class="ml-16 mr-16 mt-2 border-4 bg-slate-50/80 rounded-md border-neutral-800 divide-slate-400 divide-x-2 divide-y-2 shadow grid grid-cols-5"
        >
            <div class="col-span-1 p-1">
                <div class="text-[10px] mt-2 text-black mb-1 font-['8_Bit_Hud']">Bass Synth</div>
            </div>
            <div class="col-span-1 p-1">
                <div class="text-[10px] mt-2 text-black mb-2 font-['8_Bit_Hud']">Synth 1</div>
            </div>
            <div class="col-span-1 p-1">
                <div class="text-[10px] mt-2 text-black mb-2 font-['8_Bit_Hud']">Synth 2</div>
            </div>
            <div class="col-span-1 p-1">
                <div class="text-[10px] mt-2 text-black mb-2 font-['8_Bit_Hud']">Synth 3</div>
            </div>
            <div class="col-span-1 p-1">
                <div class="text-[10px] mt-2 text-black mb-2 font-['8_Bit_Hud']">Drums</div>
            </div>
            <div class="col-span-1">
                <div
                    v-for="(note, index) in bassNotes"
                    :key="index"
                    class="text-[10px] text-black font-['8_Bit_Hud']"
                    :class="{ 'bg-neutral-600 text-sky-50': index == progressTick }"
                >
                    {{ note }}
                </div>
            </div>
            <div class="col-span-1">
                <div
                    v-for="(note, index) in synth1Notes"
                    :key="index"
                    class="text-[10px] text-black font-['8_Bit_Hud']"
                    :class="{ 'bg-neutral-600 text-sky-50': index == progressTick }"
                >
                    {{ note }}
                </div>
            </div>
            <div class="col-span-1">
                <div
                    v-for="(note, index) in synth2Notes"
                    :key="index"
                    class="text-[10px] text-black font-['8_Bit_Hud']"
                    :class="{ 'bg-neutral-600 text-sky-50': index == progressTick }"
                >
                    {{ note }}
                </div>
            </div>
            <div class="col-span-1">
                <div
                    v-for="(note, index) in synth3Notes"
                    :key="index"
                    class="text-[10px] text-black font-['8_Bit_Hud']"
                    :class="{ 'bg-neutral-600 text-sky-50': index == progressTick }"
                >
                    {{ note }}
                </div>
            </div>
            <div class="col-span-1">
                <div
                    v-for="(note, index) in drumNotes"
                    :key="index"
                    class="text-[10px] text-black font-['8_Bit_Hud']"
                    :class="{ 'bg-neutral-600 text-sky-50': index == progressTick }"
                >
                    {{ note }}
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import * as Tone from "tone";
import * as TotalSerialism from "total-serialism";

const reverb = new Tone.Reverb().toDestination();
const vibrato = new Tone.Vibrato({
    frequency: 5,
    depth: 0.2,
}).toDestination();

const Rand = TotalSerialism.Stochastic;
const TL = TotalSerialism.Translate;
const Algo = TotalSerialism.Algorithmic;

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
    pitchDecay: 0.2,
    octaves: 12,
    oscillator: {
        type: "sawtooth",
    },
    envelope: {
        attack: 0.01,
        decay: 0.001,
        sustain: 1,
        release: 0.001,
    },
    volume: -18,
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
    volume: -30,
}).toDestination();

const synth3 = new Tone.Synth({
    oscillator: {
        type: "square",
    },
    envelope: {
        release: 0.07,
    },
    volume: -20,
}).toDestination();

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
    ["IV", "V", "III", "VI"],
    ["IVmaj7", "III7", "II7", "Imaj7"],
    ["Imin", "VI", "III", "IVmin"],
];

const PROGRESSIONSCALES = [
    [SCALES["MAJOR"], SCALES["MAJOR"], SCALES["MAJOR"], SCALES["MAJOR"]],
    [SCALES["MAJOR"], SCALES["MAJOR"], SCALES["MAJOR"], SCALES["MAJOR"]],
    [SCALES["MINOR"], SCALES["MAJOR"], SCALES["MAJOR"], SCALES["MINOR"]],
];

const FIFTHS = [60, 67, 62, 69, 64, 71, 66, 61, 68, 63, 70, 65];

const CHORDS = ["IV", "V", "III", "VI"];

let bassNotes,
    synth1Notes,
    synth2Notes,
    synth3Notes,
    drumNotes,
    kickPattern,
    hihatPattern,
    snarePattern,
    bassMelody,
    synth1Melody,
    synth2Melody,
    synth3Melody = [];
let currentChords = [];
let fifthIndex = 0;
let root = 0;
let synthRoot = root;
let bassRoot = root - 24;
let bpm = 130;
let progressionIndex = 0;
let cAutomaton = new Algo.Automaton();

// Synths Parts
const bassPart = new Tone.Part((time, value) => {
    bassSynth.triggerAttackRelease(value.note, "16n", time, value.velocity);
}).start("+0.1");

const synth1Part = new Tone.Part((time, value) => {
    synth.triggerAttackRelease(value.note, "16n", time, value.velocity);
}).start("+0.1");

const synth2Part = new Tone.Part((time, value) => {
    synth2.triggerAttackRelease(value.note, "32n", time, value.velocity);
}).start("+0.1");

const synth3Part = new Tone.Part((time, value) => {
    synth3.triggerAttackRelease(value.note, value.duration, time, value.velocity);
}).start("+0.1");

const kickPart = new Tone.Part((time, value) => {
    kickSynth.triggerAttackRelease(value.note, "32n", time, value.velocity);
}).start("+0.1");

const hhPart = new Tone.Part((time, value) => {
    hihatSynth.triggerAttackRelease("64n", time, value.velocity);
}).start("+0.1");

const snarePart = new Tone.Part((time, value) => {
    snareSynth.triggerAttackRelease("32n", time, value.velocity);
}).start("+0.1");

function fillDrums() {
    let seconds = Tone.Time("16n").toSeconds();
    kickPattern = [];
    hihatPattern = [];
    snarePattern = [];
    drumNotes = [];
    let time = 0;

    kickPart.clear();
    hhPart.clear();
    snarePart.clear();
    if (Rand.random(1, 1, 4)[0] != 1) {
        for (let i = 0; i < 128; i++) {
            if (i % 8 == 0 || i == 0) {
                kickPattern.push({ time: time, note: 20, velocity: 1 });
                drumNotes.push("KCK");
            } else if (i % 8 == 4) {
                snarePattern.push({ time: time, note: 60, velocity: 1 });
                drumNotes.push("SNR");
            } else if (i % 8 == 2) {
                if (Rand.pick(1, [1, 2, 3, 4, 5]) == 1) {
                    kickPattern.push({ time: time, note: 20, velocity: 1 });
                    drumNotes.push("KCK");
                } else {
                    hihatPattern.push({ time: time, note: 30, velocity: 1 });
                    drumNotes.push("HH");
                }
            } else {
                hihatPattern.push({ time: time, note: 30, velocity: 1 });
                drumNotes.push("HH");
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
    } else {
        for (let i = 0; i < 128; i++) {
            drumNotes.push("-");
        }
    }
}

function fillMelodyArray(melody) {
    let seconds = Tone.Time("16n").toSeconds();
    let melodyArray = [];
    let time = 0;

    for (let i = 0; i < 128; i++) {
        if (melody[i] != 0) {
            melodyArray.push({
                time: time,
                duration: i % 2 == 0 ? "4n" : "32n",
                note: Tone.Frequency(melody[i], "midi").toFrequency(),
                velocity: 1,
            });
        }
        time = time + seconds;
    }

    return melodyArray;
}

function generateBassMelody() {
    bassMelody = [];
    bassNotes = [];
    bassPart.clear();

    let euclid = Algo.euclid(128, Rand.random(1, 60, 80)[0]);
    let auxIndex = 0;
    let bassPattern = Rand.random(1, 0, 1)[0];
    let note;

    if (Rand.random(1, 1, 10)[0] != 1) {
        if (bassPattern == 0) {
            //euclidean
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 32; j++) {
                    note = (bassRoot + currentChords[i][Rand.random(1, 0, currentChords[i].length)]) * euclid[auxIndex];
                    bassNotes.push(note);

                    auxIndex++;
                }
            }
        } else {
            //1,2
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 32; j++) {
                    if (auxIndex % 4 == 0) {
                        note = bassRoot + currentChords[i][0];
                    } else if (auxIndex % 2 == 0) {
                        note = bassRoot + currentChords[i][0] + 12;
                    } else {
                        note = 0;
                    }
                    bassNotes.push(note);

                    auxIndex++;
                }
            }
        }

        bassMelody = fillMelodyArray(bassNotes);

        for (let i = 0; i < bassMelody.length; i++) {
            bassPart.add(bassMelody[i]);
        }
    } else {
        for (let j = 0; j < 128; j++) {
            bassNotes.push(0);
        }

        bassMelody = fillMelodyArray(bassNotes);
    }
}

function generateSynthMelody() {
    synth1Melody = [];
    synth1Notes = [];
    synth1Part.clear();

    let note = 0;

    if (Rand.random(1, 1, 5)[0] != 1) {
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 32; j++) {
                note =
                    synthRoot -
                    12 +
                    currentChords[i][Rand.random(1, 0, currentChords[i].length)] +
                    Rand.choose(1, [0, 12, 24])[0];
                synth1Notes.push(note);
            }
        }

        synth1Melody = fillMelodyArray(synth1Notes);

        for (let i = 0; i < synth1Melody.length; i++) {
            synth1Part.add(synth1Melody[i]);
        }
    } else {
        for (let j = 0; j < 128; j++) {
            synth1Notes.push(0);
        }

        synth1Melody = fillMelodyArray(synth1Notes);
    }
}

function generateSynth2Melody() {
    synth2Melody = [];
    synth2Notes = [];
    synth2Part.clear();

    let euclid = Algo.fastEuclid(128, Rand.random(1, 60, 100));
    let auxIndex = 0;
    let note;

    if (Rand.random(1, 1, 5)[0] != 1) {
        //euclidean
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 32; j++) {
                note =
                    (synthRoot +
                        currentChords[i][0] +
                        PROGRESSIONSCALES[progressionIndex][i][
                            Rand.random(1, 0, PROGRESSIONSCALES[progressionIndex][i].length)
                        ]) *
                    euclid[auxIndex];
                synth2Notes.push(note);
                auxIndex++;
            }
        }

        synth2Melody = fillMelodyArray(synth2Notes);

        for (let i = 0; i < synth2Melody.length; i++) {
            synth2Part.add(synth2Melody[i]);
        }
    } else {
        for (let j = 0; j < 128; j++) {
            synth2Notes.push(0);
        }

        synth2Melody = fillMelodyArray(synth2Notes);
    }
}

function generateSynth3Melody() {
    synth3Melody = [];
    synth3Notes = [];
    synth3Part.clear();

    let auxIndex = 0;
    let note;
    let gen = cAutomaton.next();

    if (Rand.random(1, 1, 8)[0] != 1) {
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 32; j++) {
                if (j == 0) {
                    note = synthRoot + currentChords[i][0];
                } else {
                    note =
                        (synthRoot +
                            currentChords[i][0] +
                            PROGRESSIONSCALES[progressionIndex][i][
                                Rand.random(1, 0, PROGRESSIONSCALES[progressionIndex][i].length)
                            ]) *
                        gen[auxIndex];
                }

                synth3Notes.push(note);
                auxIndex++;
            }
        }

        synth3Melody = fillMelodyArray(synth3Notes);

        for (let i = 0; i < synth3Melody.length; i++) {
            synth3Part.add(synth3Melody[i]);
        }
    } else {
        for (let j = 0; j < 128; j++) {
            synth3Notes.push(0);
        }

        synth3Melody = fillMelodyArray(synth3Notes);
    }
}

function setup() {
    Tone.Transport.timeSignature = [4, 4];
    Tone.Transport.loop = true;
    Tone.Transport.loopStart = 0;
    Tone.Transport.loopEnd = "16m";
    bassPart.loop = true;
    bassPart.loopStart = 0;
    bassPart.loopEnd = "8m";
    synth1Part.loop = true;
    synth1Part.loopStart = 0;
    synth1Part.loopEnd = "8m";
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
    snarePart.loop = true;
    snarePart.loopStart = 0;
    snarePart.loopEnd = "8m";
}

function setRoot() {
    let upOrDown = Rand.random(1, 0, 1)[0];

    if (upOrDown == 0) {
        if (fifthIndex + 1 > FIFTHS.length) {
            fifthIndex = 0;
        } else {
            fifthIndex = fifthIndex + 1;
        }
    } else {
        if (fifthIndex - 1 < 0) {
            fifthIndex = FIFTHS.length;
        } else {
            fifthIndex = fifthIndex - 1;
        }
    }

    root = FIFTHS[fifthIndex];
    synthRoot = root;
    bassRoot = root - 36;

    progressionIndex = Rand.random(1, 0, PROGRESSIONS.length);
    currentChords = TL.chordsFromNumerals(PROGRESSIONS[progressionIndex]);
}

function fillDisplayNotes(notes) {
    return notes.map(item => (item == 0 ? "-" : TL.midiToNote(item)));
}

function generateNewPatterns() {
    setRoot();
    generateBassMelody();
    generateSynthMelody();
    generateSynth2Melody();
    generateSynth3Melody();
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
            bassNotes: {},
            synth1Notes: {},
            synth2Notes: {},
            synth3Notes: {},
            drumNotes: {},
            progressTick: 0,
            scales: null,
            chords: null,
            modulationFrequency: 2,
            textSeed: "",
            playing: false,
            paused: false,
            root: 0,
            chords: CHORDS,
            enable1: false,
            enable2: false,
            context: {
                progression: {
                    0: "",
                    1: "",
                    2: "",
                    3: "",
                },
                root: "",
                octave: "",
            },
            rootNotes: ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"],
        };
    },
    created() {
        setup();
    },
    mounted() {
        for (let i = 0; i < 128; i++) {
            this.bassNotes[i] = "-";
            this.synth1Notes[i] = "-";
            this.synth2Notes[i] = "-";
            this.synth3Notes[i] = "-";
            this.drumNotes[i] = "-";
        }
    },
    methods: {
        changeTempo() {
            Tone.Transport.bpm.rampTo(this.tempo, 0.001);
        },
        generateText() {
            var result = "";
            var characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
            var charactersLength = characters.length;
            for (var i = 0; i < 15; i++) {
                result += characters.charAt(Math.floor(Math.random() * charactersLength));
            }
            return result;
        },
        startPlaying() {
            if (!this.playing) {
                this.playing = true;

                if (this.textSeed == "") {
                    this.textSeed = this.generateText();
                }

                Rand.seed(this.stringHash(this.textSeed));

                fifthIndex = Rand.random(1, 0, FIFTHS.length)[0];
                root = FIFTHS[fifthIndex];
        
                // feed with 40 randomly generated values 0-1
                cAutomaton.feed(Rand.coin(128));
                cAutomaton.rule(Rand.random(1, 1, 200)[0]);

                Tone.start();
                Tone.Transport.bpm.rampTo(this.tempo, 0.001);
                Tone.Transport.scheduleRepeat(time => {
                    generateNewPatterns();
                    this.changeNotes();
                }, "16m");
                Tone.Transport.scheduleRepeat(time => {
                    this.progressDisplay();
                }, "16n");
                Tone.Transport.start("+0.1");
            }
        },
        resetPlaying() {
            if (this.playing) {
                this.playing = false;
                this.paused = false;
                this.progressTick = 0;
                Tone.Transport.cancel(0);
                Tone.Transport.stop();
                for (let i = 0; i < 128; i++) {
                    this.bassNotes[i] = "-";
                    this.synth1Notes[i] = "-";
                    this.synth2Notes[i] = "-";
                    this.synth3Notes[i] = "-";
                    this.drumNotes[i] = "-";
                }
            }
        },
        stopPlaying() {
            if (this.playing) {
                this.paused = true;
                Tone.Transport.pause();
            }
        },
        resumePlaying() {
            if (this.paused) {
                this.paused = false;
                Tone.Transport.toggle();
            }
        },
        stringHash(str) {
            // https://stackoverflow.com/questions/521295/seeding-the-random-number-generator-in-javascript
            let h1 = 1779033703,
                h2 = 3144134277,
                h3 = 1013904242,
                h4 = 2773480762;

            for (let i = 0, k; i < str.length; i++) {
                k = str.charCodeAt(i);
                h1 = h2 ^ Math.imul(h1 ^ k, 597399067);
                h2 = h3 ^ Math.imul(h2 ^ k, 2869860233);
                h3 = h4 ^ Math.imul(h3 ^ k, 951274213);
                h4 = h1 ^ Math.imul(h4 ^ k, 2716044179);
            }
            h1 = Math.imul(h3 ^ (h1 >>> 18), 597399067);
            h2 = Math.imul(h4 ^ (h2 >>> 22), 2869860233);
            h3 = Math.imul(h1 ^ (h3 >>> 17), 951274213);
            h4 = Math.imul(h2 ^ (h4 >>> 19), 2716044179);

            return ((((h1 ^ h2 ^ h3 ^ h4) >>> (0 * (h2 ^ h1))) >>> (0 * (h3 ^ h1))) >>> (0 * (h4 ^ h1))) >>> 0;
        },
        changeNotes() {
            this.bassNotes = fillDisplayNotes(bassNotes);
            this.synth1Notes = fillDisplayNotes(synth1Notes);
            this.synth2Notes = fillDisplayNotes(synth2Notes);
            this.synth3Notes = fillDisplayNotes(synth3Notes);
            this.drumNotes = drumNotes;
        },
        changeTempo() {
            Tone.Transport.bpm.rampTo(this.tempo, 0.001);
        },
        progressDisplay() {
            this.progressTick == 128 ? (this.progressTick = 0) : (this.progressTick = this.progressTick + 1);
        },
        setModulation() {
            synth3.oscillator.modulationFrequency.value = this.modulationFrequency;
        },
    },
};
</script>
<style scoped>
.menuTitle {
    font-family: "Karma";
}
</style>
