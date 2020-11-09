<template>
    <div class="weapon__wrap">
        <div v-for="n in this.pages" class="page">
            <weaponCard v-for="weapon, i in weapons.slice((n - 1) * 8, (n-1) * 8 + 8)" :weapon="weapon" :key="i"></weaponCard>
        </div>
    </div>
</template>
<script lang="ts">
import weapon from './weapon.json';
import { Component, Vue, Watch } from 'vue-property-decorator';
import Chance from 'chance';


import weaponCard from './components/weaponCard.vue';
import { Weapon } from './interfaces.ts';

let chance = new Chance();

@Component({
    components: {
        weaponCard
    }
})

export default class App extends Vue {
    // declare initial data types here
    firingMechanisms: object[];
    reloadingMechanisms: object[];
    ammunition: object[];
    quirks: object[];
    sizes: object[];
    weapon: Weapon;
    weapons: Weapon[];
    pages: number;
    stepDownArr: string[];

    constructor() {
        super();

        // here is where we initialize data; not with their starting data but with empty versions of their types
        this.firingMechanisms = [];
        this.reloadingMechanisms = [];
        this.ammunition = [];
        this.quirks = [];
        this.sizes = [];
        this.weapon = {
            firingMechanism: null,
            reloadingMechanism: null,
            ammunition: null,
            quirk: null,
            size: null,
            tags: [],
            damage: null,
            damageType: null,
            notches: 0,
            name: null,
            slots: 0,
            range: '',
            bonus: ''
        };
        this.weapons = [];
        this.pages = 0;
        this.stepDownArr = ['1', '2', '1d4', '1d6', '1d8', '2d4', '1d10', '1d6 + 1d4', '1d12', '2d6', '1d8 + 1d6', '2d8', '1d10 + 1d8', '2d10']
    }

    created() {
        // we can initialize starting data values here

        let w = weapon.weaponOptions;

        this.firingMechanisms = w.firingMechanism;
        this.reloadingMechanisms = w.reloadingMechanism;
        this.ammunition = w.ammunition;
        this.quirks = w.quirk;
        this.sizes = w.size;
        this.pages = 4;
        this.weapon.name = '';
        this.weapon.slots = 0;
        this.weapon.range = ''
    }

    mounted() {
        for (let i = 0; i < 64; ++i) {
            this.createWeapon()
        }

        this.pages = this.weapons.length / 8
    }

    //methods

    createWeapon() {
        this.getAmmunition();
        this.getSize();
        this.getFiringMechanism();
        this.getReloadingMechanism();
        this.getQuirk();
        this.getName();

        this.weapons.push(this.weapon);

        this.resetWeapon();
    }

    roll(sides) {
        return chance[`d${sides}`]() - 1;
    }

    weightedRollSix() {
        let arr = [5, 4, 3, 2, 1, 0];
        let weights = [0.5, 1, 2, 3, 3, 2];

        return chance.weighted(arr, weights)
    }

    weightedRollFour() {
        let arr = [3, 2, 1, 0];
        let weights = [0.5, 1, 2, 1.5];

        return chance.weighted(arr, weights)
    }

    weightedRollEight() {
        let arr = [7, 6, 5, 4, 3, 2, 1, 0];
        let weights = [0.5, 1, 2, 2, 3, 1, 3, 1]

        return chance.weighted(arr, weights)
    }

    weightedRollTwelve() {
        let arr = [11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0];
        let weights = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 5];
        return chance.weighted(arr, weights);
    }

    getQuirk() {
        let q = this.quirks[this.weightedRollTwelve()];
        // broken by design reduces damage by a die type, so we use this to parse that

        if (q['type'] === 'broken by design') {
            let dmg = this.weapon.damage
            let i = this.stepDownArr.indexOf(dmg)
            let p = i - 1;

            this.weapon.damage = this.stepDownArr[p];
        }

        this.weapon.quirk = q;
    }

    getAmmunition() {
        // we use a weighted roll here with an emphasis on more easily attainable ammunition.
        let a = this.ammunition[this.weightedRollSix()];
        this.weapon.ammunition = a['ammoType'];
        this.weapon.damage = a['damage'];
        this.weapon.tags = [...a['tags']];
        this.weapon.damageType = a['damageType']
    }

    getName() {
        // selects a name for the weapon based on its components
        let f = this.weapon.firingMechanism;
        let a = this.weapon.ammunition;
        let r = this.weapon.reloadingMechanism;
        let s = this.weapon.slots;

        if (a === 'scrap shot') {
            // scrap shot weapons are basically useless, especially if they aren't explosively propelled, so we randomly generate a name that denotes that
            if (f !== 'explosive') {
                let modArr = ['', `lil'`, `ol'`, 'mr.', 'dr.', 'mrs.']
                let nameArr = ['inconvenience', 'frustrater', 'scrappy', 'pea shooter', 'defiable', 'insult', 'annoyance', 'irritation', 'nuisance', 'pest', 'bother', 'desperation']
                this.weapon.name = `${modArr[Math.floor(Math.random() * modArr.length)]} ${nameArr[Math.floor(Math.random() * nameArr.length)]}`
            } else {
                this.weapon.name = 'scrap shooter'
            }

        } else if (a === 'wood bolts') {
            if (r === 'manual' && f === 'tension') {
                this.weapon.name = 'shortbow'
            } else if (f !== 'tension') {
                this.weapon.name = 'dart gun'
            } else if (f === 'tension') {
                this.weapon.name = 'light crossbow'
            }

        } else if (a === 'metal bolts') {
            if (f === 'tension') {
                this.weapon.name = 'crossbow'
            } else {
                this.weapon.name = 'bolt thrower'
            }

        } else if (a === 'metal slugs') {
            if (f === 'explosive') {
                if (s < 3) {
                    if (r === 'cylinder') {
                        this.weapon.name = 'revolver'
                    } else {
                        this.weapon.name = 'pistol'
                    }
                } else {
                    if (r === 'auto') {
                        this.weapon.name = 'assault rifle'
                    } else {
                        this.weapon.name = 'battle rifle'
                    }
                }
            } else if (f === 'tension' && r === 'manual') {
                this.weapon.name = 'slingshot'
                this.weapon.damageType = 'bludgeoning'
            } else {
                this.weapon.name = 'slug slinger'
            }
        } else if (a === 'heavy shot') {
            if (f === 'explosive' && r === 'pump') {
                this.weapon.name = 'shotgun'
            } else if (s < 3) {
                this.weapon.name = 'scattergun'
            } else if (s >= 3) {
                this.weapon.name = 'scatter rifle'
            }

        } else if (a === 'explosive') {
            this.weapon.name = 'grenade launcher'
        }
    }

    getFiringMechanism() {
        let fM = this.firingMechanisms[this.weightedRollFour()];
        let mod = fM['mod'];

        // if the firing mechanism is volatile
        // or the firing mechanism is explosive and the ammunition is wood bolts
        // or there are speed or sound tags conflicting with the selected one
        // exit and run this function again

        if ((this.checkTagsForVolatile(fM['tags']) || (fM['type'] === 'explosive' && this.weapon.ammunition === 'wood bolts')) || this.checkTagsForSpeed(fM['tags']) || this.checkTagsForLoud(fM['tags'])) {
            return this.getFiringMechanism();
        }

        this.addTags(fM['tags']);

        if (mod === 'double') {
            // if the weapon damage mod is double, replace 1d# with 2d#
            let d = this.weapon.damage;
            let str = d.split('');
            str[0] = "2";

            this.weapon.damage = `${str.join('')}`;
        } else {
            // otherwise set the weapon bonus to the modifier
            this.weapon.bonus = fM['mod']
        }

        // reduce the range for shot weapons
        if (this.weapon.ammunition === 'heavy shot' || this.weapon.ammunition === 'scrap shot') {
            this.weapon.range = '30/60'
        } else {
            this.weapon.range = fM['range']
        }

        this.weapon.firingMechanism = fM['type'];
    }

    getReloadingMechanism() {
        let rM = this.reloadingMechanisms[this.weightedRollEight()];

        if (this.checkTagsForVolatile(rM['tags'])) {
            return this.getReloadingMechanism();
        }

        this.addTags(rM['tags']);

        this.weapon.reloadingMechanism = rM['type'];
    }

    getSize() {

        let arr = [3, 2, 1, 0];
        let weights = [0.5, 2, 2, 1];
        let roll = chance.weighted(arr, weights)

        let s = this.sizes[roll];

        if (this.checkTagsForVolatile(s['tags'])) {
            return this.getSize();
        }

        if (this.checkTagsForSpeed(s['tags'])) {
            return this.getSize()
        }

        if (s['slots'] > 2 && (this.weapon.ammunition === 'scrap shot' || this.weapon.ammunition === 'wood bolts')) {
            return this.getSize()
        }

        this.addTags(s['tags']);

        this.weapon.size = s['size'];
        this.weapon.slots = s['slots'];
        this.weapon.range = s['range'];
    }

    checkTagsForVolatile(tags) {
        if (tags.includes('volatile') && this.weapon['tags'].includes('volatile')) {
            return true;
        }
    }

    checkTagsForSpeed(tags) {
        if (tags.includes('fast') && this.weapon['tags'].includes('slow')) {
            return true;
        } else if (tags.includes('slow') && (this.weapon['tags'].includes('fast') || this.weapon['tags'].includes('very slow'))) {
            return true;
        }
    }

    checkTagsForLoud(tags) {
        if (tags.includes('loud') && this.weapon.tags.includes('silent')) {
            return true;
        } else if (tags.includes('silent') && this.weapon.tags.includes('loud')) {
            return true;
        }
    }

    addTags(tags) {
        for (let i = 0, l = tags.length; i < l; ++i) {
            let t = tags[i];

            if (!this.weapon.tags.includes(t) && t !== "") {
                this.weapon.tags.push(t);
            }
        }
    }

    resetWeapon() {
        this.weapon = {
            firingMechanism: null,
            reloadingMechanism: null,
            ammunition: null,
            quirk: null,
            size: null,
            tags: null,
            damage: null,
            damageType: null,
            notches: null,
            name: '',
            slots: 0,
            range: '',
            bonus: ''
        }
    }
}

</script>
