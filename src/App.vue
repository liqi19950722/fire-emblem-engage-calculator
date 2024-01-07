<script setup lang="ts">
import { reactive, ref, watch } from "vue";
import Decimal from "decimal.js";
import { cloneDeep } from "lodash";
import characterData from "./assets/data/character.json";
import classData from "./assets/data/class.json";
class Character {
    avatar: string;
    name: string;
    level: Level;
    class: Class;
    currentAttribute: Attribute;
    growthRate: Attribute;
    upperLimitCorrectedValue: Attribute;

    constructor(characterData: CharacterData, classData: ClassData) {
        this.avatar = new URL(
            "./assets/avatar/" + characterData.avatar,
            import.meta.url
        ).href;
        this.name = characterData.name;
        let { showLevel, actualLevel } = characterData.level;
        this.level = new Level(
            showLevel,
            actualLevel,
            classData.classType === ClassType.SPECIAL ? 40 : 20
        );
        let {
            name,
            classType,
            attributeCorrectedValue,
            growthRate,
            highestAttribute,
        } = classData;
        this.class = new Class(
            name,
            classType as ClassType,
            growthRate,
            attributeCorrectedValue,
            highestAttribute
        );
        this.currentAttribute = characterData.initAttribute;
        this.growthRate = characterData.growthRate;
        this.upperLimitCorrectedValue = characterData.upperLimitCorrectedValue;

        initAttribute(this);
    }

    isJean(): boolean {
        return this.name === "贾恩";
    }
}
type CharacterData = {
    avatar: string;
    name: string;
    className: string;
    level: { showLevel: number; actualLevel: number };
    initAttribute: Attribute;
    growthRate: Attribute;
    upperLimitCorrectedValue: Attribute;
};
type ClassData = {
    name: string;
    classType: string;
    attributeCorrectedValue: Attribute;
    growthRate: Attribute;
    highestAttribute: Attribute;
};
class Level {
    constructor(
        public showLevel: number,
        public actualLevel: number,
        public maxLevel: number
    ) {}
}

type Attribute = {
    hp: number;
    str: number;
    mag: number;
    dex: number;
    spd: number;
    lck: number;
    def: number;
    res: number;
    bld: number;
};

class Class {
    constructor(
        public name: string,
        public classType: ClassType,
        public growthRate: Attribute,
        public attributeCorrectedValue: Attribute,
        public highestAttribute: Attribute
    ) {}
}

enum ClassType {
    LOW_LEVEL = "LOW_LEVEL",
    HIGH_LEVEL = "HIGH_LEVEL",
    SPECIAL = "SPECIAL",
}

const reset = () => {
    const selfCharacter = initChrarcter(selectedCharacterName.value);
    Object.assign(character, selfCharacter);
    selectedClassName.value = selfCharacter.class.name;
};

const initAttribute = (character: Character): void => {
    character.currentAttribute.hp = parseFloat(
        new Decimal(character.currentAttribute.hp)
            .add(new Decimal(character.growthRate.hp).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.str = parseFloat(
        new Decimal(character.currentAttribute.str)
            .add(new Decimal(character.growthRate.str).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.mag = parseFloat(
        new Decimal(character.currentAttribute.mag)
            .add(new Decimal(character.growthRate.mag).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.dex = parseFloat(
        new Decimal(character.currentAttribute.dex)
            .add(new Decimal(character.growthRate.dex).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.spd = parseFloat(
        new Decimal(character.currentAttribute.spd)
            .add(new Decimal(character.growthRate.spd).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.lck = parseFloat(
        new Decimal(character.currentAttribute.lck)
            .add(new Decimal(character.growthRate.lck).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.def = parseFloat(
        new Decimal(character.currentAttribute.def)
            .add(new Decimal(character.growthRate.def).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.res = parseFloat(
        new Decimal(character.currentAttribute.res)
            .add(new Decimal(character.growthRate.res).dividedBy(100))
            .toFixed(2)
    );
    character.currentAttribute.bld = parseFloat(
        new Decimal(character.currentAttribute.bld)
            .add(new Decimal(character.growthRate.bld).dividedBy(100))
            .toFixed(2)
    );
};

const levelUp = (character: Character, enableStarsphere: boolean): void => {
    function judgeLevelUp() {
        return (
            character.level.showLevel >= character.level.maxLevel ||
            character.level.maxLevel >= 41
        );
    }
    doLevelUp(judgeLevelUp, enableStarsphere, character, (_, startLevel) => 1);
};

const levelTo10 = (character: Character, enableStarsphere: boolean): void => {
    function judgeLevelUp() {
        return character.level.showLevel >= 10;
    }

    function calculateUpLevel(character: Character, startLevel: number) {
        return 10 - startLevel;
    }
    doLevelUp(judgeLevelUp, enableStarsphere, character, calculateUpLevel);
};

const levelToMax = (character: Character, enableStarsphere: boolean): void => {
    function judgeLevelUp() {
        return (
            character.level.showLevel >= character.level.maxLevel ||
            character.level.actualLevel >= 41
        );
    }

    function calculateUpLevel(character: Character, startLevel: number) {
        return character.level.actualLevel > 20
            ? 40 - character.level.actualLevel
            : character.level.maxLevel - startLevel;
    }

    doLevelUp(judgeLevelUp, enableStarsphere, character, calculateUpLevel);
};

function doLevelUp(
    cantLevelUp: () => boolean,
    enableStarsphere: boolean,
    character: Character,
    calculateUpLevel: (character: Character, startLevel: number) => number
) {
    if (cantLevelUp()) {
        return;
    }
    let startLevel = character.level.showLevel;
    let upLevel = calculateUpLevel(character, startLevel);

    calculateAttributeGrowth(
        character,
        upLevel,
        getAttributeGrowthArray(
            enableStarsphere ? 15 : 0,
            character.isJean() ? 2 : 1,
            character.growthRate,
            character.class.growthRate
        )
    );
    fixUpperLimitCorrected(character);
    character.level.showLevel += upLevel;
    character.level.actualLevel += upLevel;
}

function getAttributeGrowthArray(
    starsphere: number,
    isDouble: number,
    characterGrowthRate: Attribute,
    classGrowthRate: Attribute
) {
    return {
        hp: characterGrowthRate.hp + starsphere + classGrowthRate.hp * isDouble,
        str:
            characterGrowthRate.str +
            starsphere +
            classGrowthRate.str * isDouble,
        mag:
            characterGrowthRate.mag +
            starsphere +
            classGrowthRate.mag * isDouble,
        dex:
            characterGrowthRate.dex +
            starsphere +
            classGrowthRate.dex * isDouble,
        spd:
            characterGrowthRate.spd +
            starsphere +
            classGrowthRate.spd * isDouble,
        lck:
            characterGrowthRate.lck +
            starsphere +
            classGrowthRate.lck * isDouble,
        def:
            characterGrowthRate.def +
            starsphere +
            classGrowthRate.def * isDouble,
        res:
            characterGrowthRate.res +
            starsphere +
            classGrowthRate.res * isDouble,
        bld:
            characterGrowthRate.bld +
            starsphere +
            classGrowthRate.bld * isDouble,
    };
}
function calculateAttributeGrowth(
    { currentAttribute }: Character,
    upLevel: number,
    { hp, str, mag, dex, spd, lck, def, res, bld }: Attribute
) {
    currentAttribute.hp = parseFloat(
        new Decimal(currentAttribute.hp)
            .add(new Decimal(hp).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.str = parseFloat(
        new Decimal(currentAttribute.str)
            .add(new Decimal(str).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.mag = parseFloat(
        new Decimal(currentAttribute.mag)
            .add(new Decimal(mag).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.dex = parseFloat(
        new Decimal(currentAttribute.dex)
            .add(new Decimal(dex).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.spd = parseFloat(
        new Decimal(currentAttribute.spd)
            .add(new Decimal(spd).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.lck = parseFloat(
        new Decimal(currentAttribute.lck)
            .add(new Decimal(lck).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.def = parseFloat(
        new Decimal(currentAttribute.def)
            .add(new Decimal(def).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.res = parseFloat(
        new Decimal(currentAttribute.res)
            .add(new Decimal(res).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
    currentAttribute.bld = parseFloat(
        new Decimal(currentAttribute.bld)
            .add(new Decimal(bld).mul(upLevel).dividedBy(100))
            .toFixed(2)
    );
}

function fixUpperLimitCorrected({
    currentAttribute,
    class: { highestAttribute },
    upperLimitCorrectedValue,
}: Character) {
    currentAttribute.hp = Math.min(
        currentAttribute.hp,
        highestAttribute.hp + upperLimitCorrectedValue.hp
    );
    currentAttribute.str = Math.min(
        currentAttribute.str,
        highestAttribute.str + upperLimitCorrectedValue.str
    );
    currentAttribute.mag = Math.min(
        currentAttribute.mag,
        highestAttribute.mag + upperLimitCorrectedValue.mag
    );
    currentAttribute.dex = Math.min(
        currentAttribute.dex,
        highestAttribute.dex + upperLimitCorrectedValue.dex
    );
    currentAttribute.spd = Math.min(
        currentAttribute.spd,
        highestAttribute.spd + upperLimitCorrectedValue.spd
    );
    currentAttribute.lck = Math.min(
        currentAttribute.lck,
        highestAttribute.lck + upperLimitCorrectedValue.lck
    );
    currentAttribute.def = Math.min(
        currentAttribute.def,
        highestAttribute.def + upperLimitCorrectedValue.def
    );
    currentAttribute.res = Math.min(
        currentAttribute.res,
        highestAttribute.res + upperLimitCorrectedValue.res
    );
    currentAttribute.bld = Math.min(
        currentAttribute.bld,
        highestAttribute.bld + upperLimitCorrectedValue.bld
    );
}
function calculateAttributeCorrected(
    { currentAttribute }: Character,
    { hp, str, mag, dex, spd, lck, def, res, bld }: Attribute
) {
    currentAttribute.hp = currentAttribute.hp + hp;
    currentAttribute.str = currentAttribute.str + str;
    currentAttribute.mag = currentAttribute.mag + mag;
    currentAttribute.dex = currentAttribute.dex + dex;
    currentAttribute.spd = currentAttribute.spd + spd;
    currentAttribute.lck = currentAttribute.lck + lck;
    currentAttribute.def = currentAttribute.def + def;
    currentAttribute.res = currentAttribute.res + res;
    currentAttribute.bld = currentAttribute.bld + bld;
}
function getAttributeCorrectedArray(current: Class, target: Class) {
    return {
        hp:
            target.attributeCorrectedValue.hp -
            current.attributeCorrectedValue.hp,
        str:
            target.attributeCorrectedValue.str -
            current.attributeCorrectedValue.str,
        mag:
            target.attributeCorrectedValue.mag -
            current.attributeCorrectedValue.mag,
        dex:
            target.attributeCorrectedValue.dex -
            current.attributeCorrectedValue.dex,
        spd:
            target.attributeCorrectedValue.spd -
            current.attributeCorrectedValue.spd,
        lck:
            target.attributeCorrectedValue.lck -
            current.attributeCorrectedValue.lck,
        def:
            target.attributeCorrectedValue.def -
            current.attributeCorrectedValue.def,
        res:
            target.attributeCorrectedValue.res -
            current.attributeCorrectedValue.res,
        bld:
            target.attributeCorrectedValue.bld -
            current.attributeCorrectedValue.bld,
    };
}
function judgeClassChange(current: Class, target: Class, character: Character) {
    let cantTurnTo =
        (current.classType === ClassType.LOW_LEVEL &&
            character.level.showLevel < 10 &&
            target.classType === ClassType.HIGH_LEVEL) ||
        (current.classType === ClassType.SPECIAL &&
            target.classType === ClassType.HIGH_LEVEL &&
            character.level.showLevel < 21) ||
        (current.name === target.name &&
            character.level.showLevel !== character.level.maxLevel);
    let targetShowLevel =
        target.classType === ClassType.SPECIAL &&
        (current.classType === ClassType.HIGH_LEVEL ||
            character.level.showLevel >= 21)
            ? 21
            : 1;
    let targetMaxShowLevel = target.classType === ClassType.SPECIAL ? 40 : 20;

    return {
        canTurnTo: !cantTurnTo,
        targetShowLevel: targetShowLevel,
        targetMaxShowLevel: targetMaxShowLevel,
    };
}
const changeClass = (className: string): boolean => {
    const currentClass = character.class;
    const targetClass = classDataArray.value.filter(
        (item) => item.name === className
    )[0] as Class;

    let { canTurnTo, targetShowLevel, targetMaxShowLevel } = judgeClassChange(
        currentClass,
        targetClass,
        character
    );

    if (canTurnTo) {
        character.class = targetClass;
        character.level.showLevel = targetShowLevel;
        character.level.maxLevel = targetMaxShowLevel;
        calculateAttributeCorrected(
            character,
            getAttributeCorrectedArray(currentClass, targetClass)
        );
        fixUpperLimitCorrected(character);
    }
    return canTurnTo;
};
const enableStarsphere = ref<boolean>(false);
const characterDataArray = ref(
    Array.from(Object.values(characterData)).map((value) => value)
);
const classDataArray = ref(
    Array.from(Object.values(classData)).map((value) => value)
);
const selectedCharacterName = ref<string>(characterDataArray.value[0].name);
const selectedClassName = ref<string>(classDataArray.value[0].name);
const character = reactive<Character>(
    initChrarcter(selectedCharacterName.value)
);

watch(selectedCharacterName, (newCharacterName, oldCharacterName) => {
    const newCharacter = initChrarcter(newCharacterName);
    Object.assign(character, newCharacter);

    selectedClassName.value = newCharacter.class.name;
});
watch(selectedClassName, (newClassName, oldClassName) => {
    changeClass(newClassName);
});
function initChrarcter(characterName: string): Character {
    let selectedCharacter = characterDataArray.value.filter(
        (item) => item.name === characterName
    )[0] as CharacterData;
    let selectedClass = classDataArray.value.filter(
        (item) => item.name === selectedCharacter.className
    )[0] as ClassData;
    return new Character(
        cloneDeep(selectedCharacter),
        cloneDeep(selectedClass)
    );
}
</script>

<template>
    <div class="container m-auto text-lg font-serif">
        <div class="flex justify-center mt-10">
            <div class="flex w-1/4 gap-2 shadow-md rounded-lg p-2">
                <img
                    class="object-cover object-top w-28 h-28"
                    alt="头像"
                    :src="character.avatar"
                />
                <div class="flex-grow flex flex-col justify-around">
                    <select
                        class="appearance-none text-center"
                        v-model="selectedCharacterName"
                    >
                        <option disabled>请选择一名角色</option>
                        <option v-for="item in characterDataArray" >
                            {{ item.name }}
                        </option>
                    </select>
                    <div class="flex justify-between">
                        <p class="text-left">
                            Lv. {{ character.level.showLevel }}/{{
                                character.level.maxLevel
                            }}
                        </p>
                        <p class="text-right">
                            总Lv. {{ character.level.actualLevel }}
                        </p>
                    </div>
                    <div class="flex justify-between">
                        <select
                            class="w-28 appearance-none text-center text-ellipsis"
                            v-model="selectedClassName"
                            v-show="character.class.name"
                        >
                            <option disabled>请选择职业</option>
                            <option v-for="item in classDataArray">
                                {{ item.name }}
                            </option>
                        </select>
                        <button
                            class="flex-grow text-center"
                            @click="changeClass(selectedClassName)"
                        >
                            横转
                        </button>
                    </div>
                </div>
                <div class="flex items-center justify-around flex-col">
                    <button @click="reset()">重置</button>
                    <button @click="levelUp(character, enableStarsphere)">
                        升级
                    </button>
                    <button @click="levelTo10(character, enableStarsphere)">
                        升到10级
                    </button>
                    <button @click="levelToMax(character, enableStarsphere)">
                        升到满级
                    </button>
                </div>
            </div>
        </div>
        <div class="flex justify-center mt-5">
            <div class="w-1/4 shadow-md rounded-lg p-2">
                <div class="flex justify-between items-center">
                    <span>属性</span>
                    <div class="flex gap-1">
                        <input type="checkbox" v-model="enableStarsphere" />
                        <p>星玉加护</p>
                    </div>
                    <!--  -->
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>HP</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.hp ===
                                character.upperLimitCorrectedValue.hp +
                                    character.class.highestAttribute.hp
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.hp) }}/{{
                                character.class.highestAttribute.hp
                            }}({{
                                character.upperLimitCorrectedValue.hp
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>力量</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.str ===
                                character.upperLimitCorrectedValue.str +
                                    character.class.highestAttribute.str
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.str) }}/{{
                                character.class.highestAttribute.str
                            }}({{
                                character.upperLimitCorrectedValue.str
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>魔力</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.mag ===
                                character.upperLimitCorrectedValue.mag +
                                    character.class.highestAttribute.mag
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.mag) }}/{{
                                character.class.highestAttribute.mag
                            }}({{
                                character.upperLimitCorrectedValue.mag
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>技术</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.dex ===
                                character.upperLimitCorrectedValue.dex +
                                    character.class.highestAttribute.dex
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.dex) }}/{{
                                character.class.highestAttribute.dex
                            }}({{
                                character.upperLimitCorrectedValue.dex
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>速度</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.spd ===
                                character.upperLimitCorrectedValue.spd +
                                    character.class.highestAttribute.spd
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.spd) }}/{{
                                character.class.highestAttribute.spd
                            }}({{
                                character.upperLimitCorrectedValue.spd
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>幸运</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.lck ===
                                character.upperLimitCorrectedValue.lck +
                                    character.class.highestAttribute.lck
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.lck) }}/{{
                                character.class.highestAttribute.lck
                            }}({{
                                character.upperLimitCorrectedValue.lck
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>防守</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.def ===
                                character.upperLimitCorrectedValue.def +
                                    character.class.highestAttribute.def
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.def) }}/{{
                                character.class.highestAttribute.def
                            }}({{
                                character.upperLimitCorrectedValue.def
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>魔防</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.res ===
                                character.upperLimitCorrectedValue.res +
                                    character.class.highestAttribute.res
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.res) }}/{{
                                character.class.highestAttribute.res
                            }}({{
                                character.upperLimitCorrectedValue.res
                            }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>体格</span>
                        <span>：</span>
                    </div>
                    <div class="text-right">
                        <span
                            :class="
                                character.currentAttribute.bld ===
                                character.upperLimitCorrectedValue.bld +
                                    character.class.highestAttribute.bld
                                    ? 'text-green-400'
                                    : ''
                            "
                            >{{ Math.floor(character.currentAttribute.bld) }}/{{
                                character.class.highestAttribute.bld
                            }}({{
                                character.upperLimitCorrectedValue.bld
                            }})</span
                        >
                    </div>
                </div>
            </div>
        </div>
        <div class="flex justify-center mt-5">
            <div class="w-1/4 shadow-md rounded-lg p-2">
                <div class="flex">
                    <span>总成长率</span>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>HP</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.hp
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.hp }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>力量</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.str
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.str }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>魔力</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.mag
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.mag }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>技术</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.dex
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.dex }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>速度</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.spd
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.spd }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>幸运</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex justify-center">
                        <span class="basis-6 text-right">{{
                            character.growthRate.lck
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.lck }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>防守</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex">
                        <span class="basis-6 text-right">{{
                            character.growthRate.def
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.def }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>魔防</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex">
                        <span class="basis-6 text-right">{{
                            character.growthRate.res
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.res }})</span
                        >
                    </div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-left w-16 flex justify-between">
                        <span>体格</span>
                        <span>：</span>
                    </div>
                    <div class="text-right w-16 flex">
                        <span class="basis-6 text-right">{{
                            character.growthRate.bld
                        }}</span>
                        <span class="flex-grow text-right"
                            >({{ character.class.growthRate.bld }})</span
                        >
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
