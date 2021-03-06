<template>
    <div style="max-width: 100%">
        <md-switch v-model="saving">Auto save</md-switch>
        <md-switch v-model="preview">Preview settings before downloading</md-switch>
        <md-button @click="reset_active = true" class="md-raised md-accent">Reset all</md-button>
        <md-button @click="download" class="md-raised md-primary">Download</md-button>
		<div class="ss">
			<md-button @click="update_setting" class="md-raised md-primary">Update</md-button>
			<md-field>
				<md-input v-model="setting_string" />
				<span class="md-helper-text">Settings_string</span>
			</md-field>
		</div>
		<div class="ss">
			<md-button @click="load" class="md-raised md-primary">Load</md-button>
			<md-field>
				<label>Preset</label>
				<md-select v-model="preset">
					<md-option
						v-for="(string, name) in presets"
						:key="name"
						:value="string"
					>{{name}}</md-option>
				</md-select>
			</md-field>
		</div>
        <md-tabs class="md-primary" v-on:md-changed="changeTab">
            <md-tab :id="tab" :key="tab" :md-label="tab" v-for="tab in Object.keys(settings)" />
            <md-tab id="Help" md-label="Help" />
        </md-tabs>
        <div class="component" v-if="tab === 'Starting Inventory'">
            <div v-bind:key="tabitem" v-for="tabitem in Object.keys(items)">
                <md-switch
                    class="md-primary"
                    v-model="items_choices[tabitem].active"
                >{{items[tabitem].gui_text}}</md-switch>
                <div>
                    <md-field v-if="items_choices[tabitem].active === true">
                        <label>Items allowed</label>
                        <md-select multiple v-model="items_choices[tabitem].allow">
                            <md-option
                                :key="key"
                                :value="key"
                                v-for="key in Object.keys(items[tabitem].allow)"
                            >{{items[tabitem].allow[key]}}</md-option>
                            <md-button
                                class="md-raised md-primary"
                                v-if="items_choices[tabitem].allow.length === 0"
                                v-on:click="() => {items_choices[tabitem].allow = Object.keys(items[tabitem].allow)}"
                            >Select all</md-button>
                            <md-button
                                class="md-raised md-primary"
                                v-else
                                v-on:click="() => {items_choices[tabitem].allow = []}"
                            >Deselect all</md-button>
                        </md-select>
                    </md-field>
                    <md-field style="background: #EEE" v-else>
                        <label>Items allowed</label>
                        <md-select disabled multiple v-model="items_choices[tabitem].allow" />
                    </md-field>
                    <div style="width: 80%; margin: auto;">
                        <label>
                            Min :
                            <span
                                class="badge badge-secondary"
                            >{{ items_choices[tabitem].min }}</span>
                        </label>
                        <label>
                            Max :
                            <span
                                class="badge badge-secondary"
                            >{{ items_choices[tabitem].max }}</span>
                        </label>
                        <SliderComponent
                            :disabled="items_choices[tabitem].active !== true"
                            :maxValue="items_choices[tabitem].allow.length"
                            :minValue="0"
                            :currentMinValue="items_choices[tabitem].min"
                            :currentMaxValue="items_choices[tabitem].max"
                            v-bind:maxChange.sync="items_choices[tabitem].max"
                            v-bind:minChange.sync="items_choices[tabitem].min"
                        />
                    </div>
                </div>
            </div>
        </div>
        <div v-if="tab === 'Help'">
            <h1>
                <span class="badge badge-success">Tutorial</span>
            </h1>
            <h2>Step 1</h2>
            <p>
                For each setting that you want to randomize, select it. If the setting has a list of choices, select all
                choices that you want in the pool.
            </p>
            <h2>Step 2</h2>
            <p>Download the json file with the "Download" button.</p>
            <h2>Step 3</h2>
            <p>
                Open Roman's Fork app and select the preset with non-random settings (to set the default value to
                settings
                that you didn't randomize).
            </p>
            <h2>Step 4</h2>
            <p>Insert the json file you just downloaded into the plando file.</p>

            <h1>
                <span class="badge badge-warning">Information</span>
            </h1>
            <p>At this moment, it only works with Roman's fork.</p>
        </div>
        <div class="component">
            <div v-bind:key="setting.name" v-for="setting in settings[tab]">
                <md-switch
                    :id="setting.name"
                    class="md-primary"
                    v-model="choices[setting.name].active"
                >{{setting.gui_text}}</md-switch>
                <div>
                    <md-field
                        v-if="setting.type === 'list' && choices[setting.name].active === true"
                    >
                        <label>Settings allowed</label>
                        <md-select multiple v-model="choices[setting.name].allow">
                            <md-option
                                :key="choice"
                                :value="choice"
                                v-for="choice in Object.keys(setting.choices)"
                            >{{setting.choices[choice]}}</md-option>
                            <md-button
                                class="md-raised md-primary"
                                v-if="choices[setting.name].allow.length === 0"
                                v-on:click="() => {choices[setting.name].allow = Object.keys(setting.choices)}"
                            >Select all</md-button>
                            <md-button
                                class="md-raised md-primary"
                                v-else
                                v-on:click="() => {choices[setting.name].allow = []}"
                            >Deselect all</md-button>
                        </md-select>
                    </md-field>
                    <md-field style="background: #EEE" v-else-if="setting.type === 'list'">
                        <label>Settings allowed</label>
                        <md-select class="md-secondary" disabled />
                    </md-field>
                    <div style="width: 80%; margin: auto;" v-else-if="setting.type === 'scale'">
                        <label>
                            Min :
                            <span class="badge badge-secondary">{{ choices[setting.name].min }}</span>
                        </label>
                        <label>
                            Max :
                            <span class="badge badge-secondary">{{ choices[setting.name].max }}</span>
                        </label>
                        <SliderComponent
                            :disabled="choices[setting.name].active !== true"
                            :maxValue="setting.max"
                            :minValue="setting.min"
                            :currentMinValue="choices[setting.name].min"
                            :currentMaxValue="choices[setting.name].max"
                            v-bind:maxChange.sync="choices[setting.name].max"
                            v-bind:minChange.sync="choices[setting.name].min"
                        />
                    </div>
                </div>
            </div>
        </div>
		<md-dialog :md-active.sync="showDialogError">
            <md-dialog-title>
                <span class="badge badge-danger">Error</span>
            </md-dialog-title>

            <div class="modal-body">
                <div
                    v-bind:key="err"
                    class="alert alert-danger"
                    role="alert"
                    v-for="err in errors"
                >{{err}}</div>

                <md-dialog-actions>
                    <md-button @click="showDialogError = false" class="md-primary">Close</md-button>
                </md-dialog-actions>
            </div>
        </md-dialog>
        <md-dialog :md-active.sync="showDialogWarning">
            <md-dialog-title>
                <span class="badge badge-warning">Warning</span>
            </md-dialog-title>

            <div class="modal-body">
                <div>You selected an dependency setting ! Please check when you create a seed that :</div>
                <div
                    v-bind:key="war"
                    class="alert alert-warning"
                    role="alert"
                    v-for="war in warnings"
                >{{war.setting}} is {{war.value}}</div>

                <md-dialog-actions>
                    <md-button @click="showDialogWarning = false" class="md-primary">Close</md-button>
                </md-dialog-actions>
            </div>
        </md-dialog>
        <md-dialog :md-active.sync="showDialogPreview">
            <md-dialog-title>
                <span class="badge badge-success">Preview</span>
            </md-dialog-title>

            <div class="modal-body">
                {
                <div
                    style="margin-left: 10px"
                    v-bind:key="value"
                    v-for="(value, name) in preview_content"
                >
                    {{ choices[name].gui_text }}: {{ choices[name].type === 'list' ? choices[name].choices[value] : value
                    }}
                </div>}
                <md-dialog-actions>
                    <md-button @click="showDialogPreview = false" class="md-primary">Close</md-button>
                </md-dialog-actions>
            </div>
        </md-dialog>
        <md-dialog-confirm
            :md-active.sync="reset_active"
            @md-confirm="reset"
            md-cancel-text="Cancel"
            md-confirm-text="Yes"
            md-content="All element will be definitively removed"
            md-title="Do you want reset all settings?"
        />
    </div>
</template>

<script>
import settings from "../stores/settings.json";
import items from "../stores/starting_item.json";
import SliderComponent from "./SliderComponent";

const choices = {};

const settings_list_for_string = [];

Object.values(settings).forEach(settinglist => {
    settinglist.forEach(setting => {
        choices[setting.name] = {
            type: setting.type,
            active: false,
            allow: [],
            min: setting.type === "scale" ? setting.min : undefined,
            max: setting.type === "scale" ? setting.max : undefined,
            depend: setting.depend,
            gui_text: setting.gui_text,
            choices: setting.choices
        };
        settings_list_for_string.push({
			name: setting.name,
            type: setting.type,
            choices:
                setting.type === "list" ? Object.keys(setting.choices) : null
        });
    });
});

const choices_items = {};

Object.keys(items).forEach(itemList => {
    choices_items[itemList] = {
        active: false,
        allow: [],
        min: 0,
        max: 0
    };
});

function shuffle(a) {
    let j, x, i;
    for (i = a.length - 1; i > 0; i--) {
        j = Math.floor(Math.random() * (i + 1));
        x = a[i];
        a[i] = a[j];
        a[j] = x;
    }
    return a;
}

export default {
    name: "Settings.vue",
    data() {
        return {
            settings: settings,
            tab: "Main Rules",
            choices: choices,
            showDialogError: false,
            showDialogWarning: false,
            errors: [],
            items: items,
            items_choices: choices_items,
            warnings: [],
            saving: false,
            reset_active: false,
            preview: false,
            preview_content: "",
            showDialogPreview: false,
            array: [],
			settings_list_for_string: settings_list_for_string,
			setting_string: "",
			presets: {
				"Easy" : "S99HAAAAVAAANC8AA29BQ899AAAA8AACJBEVAAAAA89DAAAAAA",
				"Standard" : "S9999BAAVAAANU9AA299R89899999AACJBEV8AAAA8999BAAAA",
				"Hard" : "S9999BAAVAAAN49AA299R89899999AACJTHZ999B98999BAAAA",
				"Hell Mode" : "99999BAAVGAAN4999999R89899999CACJ79999999899939RAA"
			},
			preset: "S99HAAAAVAAANC8AA29BQ899AAAA8AACJBEVAAAAA89DAAAAAA"
        };
    },
    methods: {
        randomItem(items) {
            if (typeof items[0] === "string")
                return items[Math.floor(Math.random() * items.length)];
            else
                return Object.keys(items)[
                    Math.floor(Math.random() * Object.keys(items).length)
                ];
        },
        randomNumber(min, max) {
            return Math.floor(Math.random() * (max - min + 1) + min);
        },
        randomBoolean() {
            return this.randomNumber(0, 1) === 0;
        },
        filter_random(list, nb) {
            const cpy = list.filter(() => true);
            shuffle(cpy);
            const output = [];
            for (let i = 0; i < nb; i++) {
                output.push(cpy.pop());
            }
            return output;
        },
        downloadObjectAsJson(exportObj, exportName) {
            const dataStr =
                "data:text/json;charset=utf-8," +
                encodeURIComponent(JSON.stringify(exportObj, null, 4));
            const downloadAnchorNode = document.createElement("a");
            downloadAnchorNode.setAttribute("href", dataStr);
            downloadAnchorNode.setAttribute("download", exportName + ".json");
            document.body.appendChild(downloadAnchorNode); // required for firefox
            downloadAnchorNode.click();
            downloadAnchorNode.remove();
        },
        changeTab(tab) {
            this.tab = tab;
        },
        generate() {
            const err = [];
            const settings = {};
            const choicesTrue = Object.keys(this.choices).filter(
                key => this.choices[key].active
            );
            choicesTrue.forEach(key => {
                const setting = this.choices[key];
                if (setting.type === "list") {
                    if (setting.allow.length === 0) {
                        err.push(
                            'The setting "' +
                                setting.gui_text +
                                '" is randomized but has no element allowed'
                        );
                    }
                    settings[key] = this.randomItem(setting.allow);
                } else if (setting.type === "boolean") {
                    settings[key] = this.randomBoolean();
                } else {
                    settings[key] = this.randomNumber(
                        setting.min,
                        setting.max + 1
                    );
                }
            });
            if (err.length > 0) {
                this.showDialogError = true;
                this.errors = err;
                return null;
            }
            const warnings = [];
            const dependencies = this.getDependencies();
            Object.keys(dependencies).forEach(settingToRemove => {
                const depend = dependencies[settingToRemove].depend;
                Object.keys(depend).forEach(key => {
                    if (
                        settings[key] !== undefined &&
                        settings[key] !== depend[key]
                    ) {
                        delete settings[settingToRemove];
                    } else if (
                        settings[key] === undefined &&
                        settings[settingToRemove] !== undefined
                    ) {
                        warnings.push({
                            setting: key,
                            value: depend[key]
                        });
                    }
                });
            });
            if (warnings.length > 0) {
                this.showDialogWarning = true;
                this.warnings = warnings;
            }
            Object.keys(this.items_choices).forEach(key => {
                if (this.items_choices[key].active === true) {
                    settings[key] = this.filter_random(
                        this.items_choices[key].allow,
                        this.randomNumber(
                            this.items_choices[key].min,
                            this.items_choices[key].max
                        )
                    );
                }
            });
            if (this.preview === true) {
                this.showDialogPreview = true;
                this.preview_content = settings;
            }
            return settings;
        },
        getDependencies() {
            const dependencies_settings = Object.keys(this.choices).filter(
                key => this.choices[key].depend !== undefined
            );
            const dependenciesObject = {};
            dependencies_settings.forEach(key => {
                dependenciesObject[key] = this.choices[key];
            });
            return dependenciesObject;
        },
        download() {
            const settings = this.generate();
            if (settings === null) {
            } else
                this.downloadObjectAsJson(
                    {
                        settings: settings
                    },
                    "settings_random"
                );
		},
		setting_to_array() {
			let array = [];
                this.settings_list_for_string.forEach(setting => {
					const setting_active = this.choices[setting.name].active;
					const setting_allow = this.choices[setting.name].allow;
                    if (setting_active) array.push(1);
                    else array.push(0);
                    if (setting.type === "list") {
                        setting.choices.forEach(choice => {
                            if (setting_allow.indexOf(choice) >= 0)
                                array.push(1);
                            else array.push(0);
                        });
					}
					else if(setting.type === 'scale') {
						const min = this.choices[setting.name].min;
						const max = this.choices[setting.name].max;
						for(let i=0; i<10; i++) {
							if(min & (1<<10-i)) array.push(1)
							else array.push(0)
						}
						for(let i=0; i<10; i++) {
							if(max & (1<<10-i)) array.push(1)
							else array.push(0)
						}
					}
                });
                this.array = array;
		},
		array_to_string() {
			let chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"
			let bits = this.array.filter(() => true);
			//pad the bits array to be multiple of 5
			if(bits.length % 5 > 0)
				for(let i =0; i<5 - bits.length % 5; i++) bits.push(0)
			//convert to characters
			let result = ""
			for(let i =0; i < bits.length; i+=5)
			{
				let value = 0
				for(let b = i; b<i+5; b++)
					value |= bits[b] << b-i
				result += chars[value]
			}
			this.setting_string = result;
		},
		string_to_array() {
			let bits = []
			let chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"
			for(let i=0; i<this.setting_string.length; i++)
			{
				let c = this.setting_string.charAt(i);
				let index = chars.indexOf(c)
				for(let b=0; b<5; b++)
				{
					bits.push((index >> b) & 1)
				}
			}
			this.array = bits;
		},
		array_to_settings() {
			this.settings_list_for_string.forEach(setting => {
				const setting_active = this.choices[setting.name].active;
				const setting_allow = this.choices[setting.name].allow;
				this.choices[setting.name].active = this.array.shift() === 1
				if (setting.type === "list") {
					setting.choices.forEach(choice => {
						if(this.array.shift() === 1) {
							this.choices[setting.name].allow.push(choice)
						}
					});
				}
				else if(setting.type === 'scale') {
					let min = 0;
					let max = 0;
					for(let i=0; i<10; i++) {
						if(this.array.shift() === 1) min |= (1<<10-i)
					}
					for(let i=0; i<10; i++) {
						if(this.array.shift() === 1) max |= (1<<10-i)
					}
					this.choices[setting.name].min = min;
					this.choices[setting.name].max = max;
				}
			});
		},
		update_setting() {
            this.reset();
			this.string_to_array();
			this.array_to_settings();
		},
		load() {
			this.setting_string = this.preset;
		},
        reset() {
            const choices_default = {};

            Object.values(settings).forEach(settinglist => {
                settinglist.forEach(setting => {
                    choices_default[setting.name] = {
                        type: setting.type,
                        active: false,
                        allow: [],
                        min: setting.type === "scale" ? setting.min : undefined,
                        max: setting.type === "scale" ? setting.max : undefined,
                        depend: setting.depend,
                        gui_text: setting.gui_text
                    };
                });
            });

            const choices_items_default = {};

            Object.keys(items).forEach(itemList => {
                choices_items_default[itemList] = {
                    active: false,
                    allow: [],
                    min: 0,
                    max: 0
                };
            });
            this.choices = choices_default;
            this.items_choices = choices_items_default;
        }
    },
    watch: {
        choices: {
            handler: function(value) {
                if (this.saving === true) {
                    const storage = {
                        choices: value,
                        items_choices: this.items_choices
                    };
                    localStorage.settings = JSON.stringify(storage);
                }

                /** Convert to settings string */
				this.setting_to_array()
				this.array_to_string()
            },
            deep: true
        },
        items_choices: {
            handler: function(value) {
                if (this.saving === true) {
                    const storage = {
                        choices: this.choices,
                        items_choices: value
                    };
                    localStorage.settings = JSON.stringify(storage);
                }
            },
            deep: true
        },
        saving: function(value) {
            if (value === false) {
                localStorage.removeItem("settings");
            } else {
                const storage = {
                    choices: this.choices,
                    items_choices: this.items_choices
                };
                localStorage.settings = JSON.stringify(storage);
            }
        }
    },
    mounted: function() {
        this.$nextTick(function() {
            if (localStorage.settings !== undefined) {
                this.saving = true;
                const storage = JSON.parse(localStorage.settings);
                this.choices = storage.choices;
                this.items_choices = storage.items_choices;
			}
			this.setting_to_array();
			this.array_to_string()
        });
    },
    components: { SliderComponent }
};
</script>

<style scoped>
@media screen and (min-width: 768px) {
    .component {
        display: grid;
        grid-auto-flow: row;
        grid-template-columns: 1fr 1fr;
        grid-column-gap: 10px;
        grid-row-gap: 15px;
        justify-content: start;
        overflow: hidden;
        min-height: 0; /* NEW */
        min-width: 0; /* NEW; needed for Firefox */
    }

    .component > div {
        overflow: hidden; /* NEW */
        min-width: 0; /* NEW; needed for Firefox */
    }
}

@media screen and (min-width: 800px) {
    .component {
        grid-template-columns: 1fr 1fr 1fr 1fr;
    }
}

.component > div {
    background: #fafafa;
}

.md-dialog {
    width: 768px;
}

.ss {
	display: flex;
}

@media screen and (max-width: 800px) {
    .ss {
        flex-direction: column;
    }
}
</style>
