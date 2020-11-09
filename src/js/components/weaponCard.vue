<template>
    <div class="weapon__card">
        <h3 class="weapon__title">{{ weapon.name }}</h3>
        <div class="split">
            <div class="stat">
                <span>Firing</span>
                <span>{{ weapon.firingMechanism }}</span>
            </div>
            <div class="stat">
                <span>Reloading</span>
                <span>{{ weapon.reloadingMechanism }}</span>
            </div>
        </div>
        <div class="split">
            <div class="stat">
                <span>Ammo</span>
                <span>{{ weapon.ammunition }}</span>
            </div>
            <div class="stat">
                <span>Damage</span>
                <span>{{ weapon.damage }} <template v-if="weapon.bonus !== ''"> + </template> {{ weapon.bonus }} {{ weapon.damageType }}</span>
            </div>
        </div>
        <div class="split">
            <div class="stat">
                <span>Size</span>
                <span>{{ weapon.size }} ({{ weapon.slots }})</span>
            </div>
            <div class="stat">
                <span>Range</span>
                <span>{{ weapon.range }}</span>
            </div>
        </div>
        <div class="stat">
            <span class="notches">Notches</span>
            <div class="notch-row">
                <span v-for="n in 5" :class="{filled : weapon.quirk.type === 'broken by design' & n < 4 }" class="notch"></span>
            </div>
        </div>
        <div class="stat quirk">
            <span>Quirk: {{ weapon.quirk.type}}</span>
            <span>{{ weapon.quirk.effect }}</span>
        </div>
        <div class="stat tags">
            <span>Tags</span>
            <div>
                <span v-for="tag in weapon.tags" class="tag">{{ tag }}</span></span>
            </div>
        </div>
        <div class="stat weapon__description">
            <span>Description/Notes</span>
        </div>
    </div>
</template>
<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';
import { Weapon } from '../interfaces.ts';

@Component
export default class WeaponCard extends Vue {
    @Prop() weapon: Weapon;

    constructor() {
        super();

    }
}

</script>
<style lang="css">
</style>
