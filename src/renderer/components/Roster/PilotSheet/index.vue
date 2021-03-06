<template>
  <div class="roster-content">
    <div v-if="pilot.name">
      <v-container fluid>
        <v-layout>
          <v-flex xs10>
            <v-layout align-end>
              <v-flex shrink>
                <editable-label attr="callsign" description="Callsign" :placeholder="pilot.callsign">
                  <span slot="label" class="display-2">{{pilot.callsign}}</span>
                </editable-label>
              </v-flex>
              <v-flex>
               <editable-label attr="name" description="Name" :placeholder="pilot.name">
                  <span slot="label" class="blockquote ml-1 pl-1">{{pilot.name}}</span>
                </editable-label>           
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex>
                  <span class="caption">HP {{stats.hp}} // ARMOR {{stats.armor}}</span>
                  <v-rating v-model="stats.hp" hover x-large :length="stats.hp + stats.armor" readonly small dense empty-icon="brightness_7" full-icon="brightness_1"/>
              </v-flex>
                <v-flex>
                  <span class="caption">E-DEFENSE {{stats.edef}}</span>
                  <v-rating v-model="stats.edef" hover x-large :length="stats.edef" readonly small dense empty-icon="brightness_7" full-icon="brightness_1"/>
                </v-flex>
                <v-flex>
                  <span class="caption">EVASION {{stats.evasion}}</span>
                  <v-rating v-model="stats.evasion" hover x-large :length="stats.evasion" readonly small dense empty-icon="brightness_7" full-icon="brightness_1"/>
                </v-flex>
                <v-flex>
                  <span class="caption">SPEED {{stats.speed}}</span>
                  <v-rating v-model="stats.speed" hover x-large :length="stats.speed" readonly small dense empty-icon="brightness_7" full-icon="brightness_1"/>
                </v-flex>
            </v-layout>
          </v-flex>
          <v-flex shrink>
            <span class="caption" style="float:right; text-align: right">LICENSE LEVEL</span><br>
          </v-flex>
            <span style="font-size: 120px; line-height: 90px; float:right; font-weight:100">{{pilot.level}}</span>
          <v-flex shrink>
            <v-tooltip bottom nudge-right="15px">
              <v-btn :to="'/level'" slot="activator" bottom right fab small :disabled="pilot.level > 11" color="primary" style="float:right; margin-left:30px">
                <v-icon large>arrow_upward</v-icon>
              </v-btn>
              <span>Level Up</span>
            </v-tooltip>
            <v-tooltip bottom nudge-right="15px">
              <v-btn slot="activator" @click="levelEditor = true" outline absolute icon small color="grey" style="top:115px; right:59px">
                <v-icon small>build</v-icon>
              </v-btn>
              <span>Set Pilot Level</span>
            </v-tooltip>
            <v-dialog v-model="levelEditor" width=900>
                <v-card>
                  <v-card-title class="title">Set Pilot Level</v-card-title>
                  <v-card-text class="text-xs-center">
                    <p><i>This tool skips the level up wizard. Triggers, licenses, talents, mech skills, and CORE bonuses will have to be updated manually</i></p>
                    <br>
                    <v-layout justify-center>
                      <v-flex xs3>
                        <v-card color="grey lighten-3" elevation=10 height=100>
                          <v-card-text>
                            <span class="title"><b class="caption">Current Level</b><br>{{pilot.level}}</span>
                          </v-card-text>
                        </v-card>
                      </v-flex>

                      <v-flex xs1 class="ml-2 mr-2">
                        <v-card flat>
                          <v-card-text class="mt-3">
                            <v-icon large>arrow_forward</v-icon>
                          </v-card-text>
                        </v-card>
                      </v-flex>
                      
                      <v-flex xs3>
                        <v-card color="grey lighten-3" elevation=10 height=100>
                          <v-card-text>
                              <v-text-field v-model="levelSkip" type="number" label="New Level" append-outer-icon="add" @click:append-outer="levelSkip ++" prepend-icon="remove" @click:prepend="levelSkip --" class="pb-0 mb-0"/>
         
                          </v-card-text>
                        </v-card>
                      </v-flex>
                    </v-layout>
                  </v-card-text>
                  <v-layout>
                    <v-flex>
                      <v-alert :value="levelSkip && levelSkip < pilot.level" type="warning">
                        <b>WARNING: LEVEL REMOVAL</b><br>This will reset all Pilot Skill Triggers, Talents, Licenses, Mech Skills, and CORE Bonuses. They must be re-added manually.
                      </v-alert>
                    </v-flex>
                  </v-layout>
                  <v-divider />
                  <v-card-actions>
                    <v-btn color="primary"  flat @click="levelEditor = false" > Cancel </v-btn>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" @click="setLevel" > Set Pilot Level</v-btn>
                  </v-card-actions>
                </v-card>
            </v-dialog>
          </v-flex>
        </v-layout>
        <v-layout v-if="!pilot.configs.length">
          <v-flex>
          <v-alert :value="!pilot.configs.length" color="info" icon="info" outline class="ma-2 ml-5 mr-5">
          <b>No Associated Mech</b><br>
          This pilot does not have any mech Configurations associated with their profile.A new Configuration can be added by navigating to the <b>MECH HANGAR</b> from the menu bar
        </v-alert>
          </v-flex>
        </v-layout>
        <v-layout>
          <v-flex xs8>
            <v-layout>
              <v-flex>
                <v-layout><span class="header no-icon">Biography</span></v-layout>
                <v-layout>
                  <v-flex xs12 style="text-align:center">
                    <b v-if="pilot.custom_background"> {{ pilot.custom_background }} </b>
                    <div v-else style="display: inline">
                      <span v-if="item('Backgrounds', pilot.background).err" class="grey--text">
                        // MISSING BACKGROUND DATA //
                      </span>
                      <b v-else> {{ item('Backgrounds', pilot.background).name }} </b>
                    </div>
                    <v-dialog lazy v-model="backgroundModal" fullscreen hide-overlay transition="dialog-bottom-transition">
                      <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary">
                        <v-icon small>edit</v-icon>
                      </v-btn>
                      <v-card>
                        <v-toolbar fixed dense flat>
                          <v-toolbar-title>Select Pilot Background</v-toolbar-title>
                          <v-spacer></v-spacer>
                          <v-toolbar-items>
                            <v-btn icon large @click="backgroundModal = false"> <v-icon large>close</v-icon> </v-btn>
                          </v-toolbar-items>
                        </v-toolbar>
                        <v-spacer></v-spacer>
                        <br><br>
                        <background-selector @selected="backgroundSelect" :preSelected="pilot.background"/>
                      </v-card>
                    </v-dialog>
                  </v-flex>
                </v-layout>
                <span class="ml-3 caption grey--text">Invocations</span>
                <v-layout wrap class="ml-3 mr-3">
                  <v-flex shrink v-for="(invoke, index) in pilot.invocations" :key="invoke.trigger">
                    <invocation-item :invoke="invoke" :index="index" @remove-invoke="removeInvocation"/>
                  </v-flex>
                  <v-flex>
                    <v-dialog v-model="invokeDialog" width="600" >
                        <v-btn slot="activator" flat small icon color="primary" :disabled="pilot.invocations && pilot.invocations.length >= 4">
                          <v-tooltip top :disabled="pilot.invocations && pilot.invocations.length >= 4">
                            <v-icon slot="activator">add_circle</v-icon>
                            <span>Add New Invocation</span>
                          </v-tooltip>
                        </v-btn>     
                      <v-card>
                        <v-card-title class="title">Add Background Invocation</v-card-title>
                        <v-card-text>
                          <v-text-field v-model="invoke_trigger" label="Invocation Trigger" outline />
                          <v-flex class="text-xs-center">
                          <v-btn-toggle v-model="invoke_attribute" dark>
                            <v-btn large color="primary">
                              <v-icon small>thumb_up</v-icon>&emsp;<span>Accuracy</span>
                            </v-btn>
                            <v-btn large color="error">
                              <v-icon small>thumb_down</v-icon>&emsp;<span>Difficulty</span>
                            </v-btn>
                          </v-btn-toggle>
                          </v-flex>
                        </v-card-text>
                        <v-divider />
                        <v-card-actions>
                          <v-btn color="primary" flat @click="invokeDialog = false"> Cancel </v-btn>
                          <v-spacer />
                          <v-btn color="primary" :disabled="invoke_trigger === '' || (!invoke_attribute && invoke_attribute !== 0)" @click="addInvocation">Add New Invocation</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </v-flex>
                </v-layout>
                <v-layout v-if="pilot.quirk">
                  <v-flex class="text-xs-center">
                    <v-alert :value="true" color="amber darken-4" class="ma-2">
                      <b style="letter-spacing: 3px; text-transform: uppercase">Clone Quirk</b>
                      <editable-label  :description="'Clone Quirk'" :attr="'quirk'" :placeholder="pilot.quirk" >
                        <span slot="label" class="p">{{pilot.quirk}}</span>
                      </editable-label>
                    </v-alert>
                  </v-flex>
                </v-layout>
                <editable-textfield :description="'History'" :attr="'history'" :initial="pilot.history" :key="pilot.id"/>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex>
                <v-layout><span class="header no-icon">Contacts</span></v-layout>
                <contacts-list :contacts="pilot.contacts" @add-contact="refresh" :key="pilot.contacts.length" />
              </v-flex>
            </v-layout>
          </v-flex>
          <v-flex xs4>
            <v-layout>
              <span class="header">Appearance
                <v-btn class="edit-btn mlneg" small flat icon color="primary" @click="appearanceModal = true; appearanceLoader = true;"><v-icon small>edit</v-icon></v-btn>
              </span>
            </v-layout>
            <v-layout>
              <v-flex class="pl-2"  @click="appearanceModal = true; appearanceLoader = true">
                <div v-if="pilot.portrait">
                  <v-img :src="`file://${userDataPath}/img/portrait/${pilot.portrait}`" max-height="55vh" max-width="45.1vw" contain/>
                </div>
                <div v-else>
                  <v-btn block small flat color="primary lighten-1"><v-icon small>add</v-icon>&nbsp;Add Pilot Image</v-btn>
                </div>
                <v-dialog lazy v-model="appearanceModal" fullscreen hide-overlay transition="dialog-bottom-transition">
                  <v-card>
                    <v-toolbar fixed dense flat>
                      <v-toolbar-title>Set Pilot Images</v-toolbar-title>
                      <v-spacer></v-spacer>
                      <v-toolbar-items>
                        <v-btn icon large @click="appearanceModal = false; appearanceLoader = false"> <v-icon large>close</v-icon> </v-btn>
                      </v-toolbar-items>
                    </v-toolbar>
                    <v-spacer class="mt-5" />
                    <div v-if="appearanceLoader">
                      <image-selector :preselectPortrait="pilot.portrait" @assign-portrait="setPortrait" />
                    </div>
                    <v-layout justify-space-between>
                      <v-flex xs1> &emsp; </v-flex>
                      <v-flex xs1><v-btn color="primary" flat @click="appearanceModal = false; appearanceLoader = false">Confirm</v-btn></v-flex>
                    </v-layout>
                  </v-card>
                </v-dialog>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex class="pl-2">
                <editable-textfield :description="'Description'" :attr="'text_appearance'" :initial="pilot.text_appearance" :key="pilot.id" />
              </v-flex>
            </v-layout>
          </v-flex>
        </v-layout>
      <v-layout>
        <v-flex xs1>
          <v-layout><span class="header no-icon">Grit</span></v-layout>
          <v-layout align-center justify-center column fill-height>
            <v-flex><span class="display-3 font-weight-black text-xs-center">+{{stats.grit}}</span></v-flex>
          </v-layout>
        </v-flex>
        <v-flex>
          <v-layout>
            <span class="header">
              Skill Triggers
              <v-dialog lazy v-model="skillModal" fullscreen hide-overlay transition="dialog-bottom-transition">
                <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary darken-2" @click="skillLoader = true">
                  <v-icon small>edit</v-icon>
                </v-btn>
                <v-card>
                  <v-toolbar fixed dense flat>
                    <v-toolbar-title>Edit Pilot Skills</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-toolbar-items>
                      <v-btn icon large @click="skillModal = false; skillLoader = false"> <v-icon large>close</v-icon> </v-btn>
                    </v-toolbar-items>
                  </v-toolbar>
                <v-spacer class="mb-4 pb-2"/>
                  <div v-if="skillLoader">
                    <skill-selector ref="skillSelector" :pilotSkills="pilot.skills" :pilotLevel="pilot.level" @set-skills="setPilotSkills" />
                  </div>
                </v-card>
              </v-dialog>
            </span>
          </v-layout>
          <div v-for="skill in pilot.skills" :key="skill.id">
            <skill-item :skillData="item('Skills', skill.id)" :skill="skill" />
          </div>
        </v-flex>
      </v-layout>
      <v-layout>
        <span class="header">Licenses
          <v-dialog lazy v-model="licenseModal" fullscreen hide-overlay transition="dialog-bottom-transition">
            <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary darken-2" @click="licenseLoader = true">
              <v-icon small>edit</v-icon>
            </v-btn>
            <v-card>
              <v-toolbar fixed dense flat>
                <v-toolbar-title>Edit Pilot Licenses</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-toolbar-items>
                  <v-btn icon large @click="licenseModal = false"> <v-icon large>close</v-icon> </v-btn>
                </v-toolbar-items>
              </v-toolbar>
              <v-spacer/>
              <div v-if="licenseLoader" class="mt-5">
                <license-selector ref="licenseSelector" :pilotLicenses="pilot.licenses" :pilotLevel="pilot.level" @set-licenses="setLicenses"/>
              </div>
              <v-layout justify-space-between>
                <v-flex xs1> &emsp; </v-flex>
                <v-flex xs1><v-btn color="primary" flat @click="licenseModal = false; licenseLoader = false">Confirm</v-btn></v-flex>
              </v-layout>
            </v-card>
          </v-dialog>
        </span>
      </v-layout>
      <div v-for="(license, index) in pilot.licenses" :key="index" class="ml-3 mr-3">
        <license-item :license="license" :licenseData="getLicense(license.name)" />
      </div>
      <v-layout>
        <span class="header">Talents
          <v-dialog lazy v-model="talentModal" fullscreen hide-overlay transition="dialog-bottom-transition">
            <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary darken-2" @click="talentLoader = true">
              <v-icon small>edit</v-icon>
            </v-btn>
            <v-card>
              <v-toolbar fixed dense flat>
                <v-toolbar-title>Edit Pilot Talents</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-toolbar-items>
                  <v-btn icon large @click="talentModal = false; talentLoader = false"> <v-icon large>close</v-icon> </v-btn>
                </v-toolbar-items>
              </v-toolbar>
              <v-spacer class="pb-5"/>
              <div v-if="talentLoader">
                <talent-selector ref="talentSelector" :pilotTalents="pilot.talents" :pilotLevel="pilot.level" @set-talents="setPilotTalents"/>
              </div>
            </v-card>
          </v-dialog>
        </span>
      </v-layout>
      <div class="ml-3 mr-3">
        <v-expansion-panel focusable>
          <talent-item v-for="talent in pilot.talents" :key="talent.id" :talent="talent" :talentData="item('Talents', talent.id)"/>
        </v-expansion-panel>
      </div>
      <v-layout>
        <span class="header">Mech Skills
          <v-dialog lazy v-model="mechSkillModal" fullscreen hide-overlay transition="dialog-bottom-transition">
            <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary darken-2" @click="mechSkillLoader = true">
              <v-icon small>edit</v-icon>
            </v-btn>
            <v-card>
              <v-toolbar fixed dense flat>
                <v-toolbar-title>Edit Mech Skills</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-toolbar-items>
                  <v-btn icon large @click="mechSkillModal = false; mechSkillLoader = false"> <v-icon large>close</v-icon> </v-btn>
                </v-toolbar-items>
              </v-toolbar>
              <v-spacer class="mb-4 pb-2"/>
              <div v-if="mechSkillLoader">
                <mech-skills-selector :mechSkills="pilot.mechSkills" :pilotLevel="pilot.level" :isActivePilot="true" />
              </div>
              <v-layout justify-space-between>
                <v-flex xs1> &emsp; </v-flex>
                <v-flex xs1><v-btn color="primary" flat @click="mechSkillModal = false; mechSkillLoader = false">Confirm</v-btn></v-flex>
              </v-layout>
            </v-card>
          </v-dialog>
        </span>
      </v-layout>
      <v-layout center justify-space-around class="pl-5">
        <v-flex>
          HULL <span class="grey--text">({{pilot.mechSkills.hull}})</span>
          <v-rating v-model="pilot.mechSkills.hull" hover x-large length=6 readonly dense empty-icon="radio_button_unchecked" full-icon="brightness_1"/>
        </v-flex>
        <v-flex xs3>
          AGILITY <span class="grey--text">({{pilot.mechSkills.agi}})</span>
          <v-rating v-model="pilot.mechSkills.agi" hover x-large length=6 readonly dense empty-icon="radio_button_unchecked" full-icon="brightness_1"/>
        </v-flex>
        <v-flex xs3>
          SYSTEMS <span class="grey--text">({{pilot.mechSkills.sys}})</span>
          <v-rating v-model="pilot.mechSkills.sys" hover x-large length=6 readonly dense empty-icon="radio_button_unchecked" full-icon="brightness_1"/>
        </v-flex>
        <v-flex xs3>
          ENGINEERING <span class="grey--text">({{pilot.mechSkills.eng}})</span>
          <v-rating v-model="pilot.mechSkills.eng" hover x-large length=6 readonly dense empty-icon="radio_button_unchecked" full-icon="brightness_1"/>
        </v-flex>
      </v-layout>
      <v-layout>
        <span class="header">CORE Bonuses
          <v-dialog lazy v-model="bonusModal" fullscreen hide-overlay transition="dialog-bottom-transition">
            <v-btn slot="activator" class="edit-btn mlneg" small flat icon color="primary darken-2" @click="cbLoader = true">
              <v-icon small>edit</v-icon>
            </v-btn>
            <v-card>
              <v-toolbar fixed dense flat>
                <v-toolbar-title>Edit CORE Bonuses</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-toolbar-items>
                  <v-btn icon large @click="bonusModal = false; cbLoader = false"> <v-icon large>close</v-icon> </v-btn>
                </v-toolbar-items>
              </v-toolbar>
              <br><br>
              <v-spacer></v-spacer>
              <div v-if="cbLoader">
                <core-bonus-selector ref="cbSelector" :pilotBonuses="pilot.core_bonuses" :pilotLevel="pilot.level" :pilotLicenses="pilot.licenses" @set-bonuses="setPilotBonuses"/>
              </div>
            </v-card>
          </v-dialog>
        </span>
      </v-layout>
      <v-container>      
      <v-layout row v-for="cb in pilot.core_bonuses" :key="cb">
        <core-bonus-item :cb="item('CoreBonuses', cb)" />
      </v-layout>
      </v-container>
      <v-layout><span class="header no-icon">Pilot Gear</span></v-layout>
      <v-layout>
        <v-flex xs12>
          <pilot-loadout />
        </v-flex>
      </v-layout>
      <v-layout><span class="header no-icon">Notes</span></v-layout>
      <v-layout>
        <v-flex>
          <editable-textfield :description="'Pilot Notes'" :attr="'notes'" :initial="pilot.notes" :key="pilot.id" />
          </v-flex>
      </v-layout>
        <v-layout justify-center>
        <v-flex xs10>
              <v-dialog lazy v-model="newConfigModal" fullscreen hide-overlay transition="fade-transition">
                <v-card>
                  <v-toolbar fixed dense flat>
                    <v-toolbar-title>New Configuration</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-toolbar-items>
                      <v-btn icon large @click="newConfigModal = false; newConfigLoader = false"> <v-icon large>close</v-icon> </v-btn>
                    </v-toolbar-items>
                  </v-toolbar>
                  <v-spacer class="ma-5" />
                  <div v-if="newConfigLoader">
                    <new-config :pilot="pilot" @close="goToConfig"/>
                  </div>
                </v-card>
            </v-dialog>
          </v-flex>
      </v-layout>
    </v-container>
      <v-divider />
        <v-layout justify-space-around fill-height class="ml-5 pl-5 mt-4 mb-4">
          <v-flex xs3><v-btn color="primary" large flat @click="openPrintOptions"><v-icon>print</v-icon>&nbsp; PRINT</v-btn ></v-flex>

          <v-flex xs3>
            <v-dialog v-model="exportDialog" width="500" >
                <v-btn slot="activator" color="primary" large flat><v-icon>call_made</v-icon> &nbsp; EXPORT</v-btn>
                <v-card>
                  <v-card-title class="title">Export Pilot &mdash; {{pilot.callsign}}</v-card-title>
                  <v-card-text>
                    <v-btn large block flat color="primary" @click="exportPilot">Save to File</v-btn>
                    <br>
                    <v-btn large block flat color="primary" @click="copyPilot">Copy Pilot Data to Clipboard</v-btn>
                  </v-card-text>
                  <v-divider />
                  <v-card-actions>
                    <v-btn color="primary"  flat @click="exportDialog = false" > Cancel </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <!-- Copy Success notification -->
              <v-snackbar v-model="snackbar" :timeout="5000" >
                <span v-html="notification" />
                <v-btn color="pink" flat @click="snackbar = false" > Close </v-btn>
              </v-snackbar>
          </v-flex>    
                
          <v-flex xs3>
            <v-btn slot="activator" color="primary" large flat @click="clonePilot"><v-icon>file_copy</v-icon> &nbsp; CLONE</v-btn>
          </v-flex>

          <v-flex xs3>
            <v-dialog v-model="deleteDialog" width="500" >
                <v-btn slot="activator" color="error" large flat><v-icon>delete</v-icon> &nbsp; DELETE</v-btn>
                <v-card>
                  <v-card-text>
                    Are you sure you want to delete {{pilot.callsign}}? This action cannot be undone
                  </v-card-text>
                  <v-divider />
                  <v-card-actions>
                    <v-btn color="primary"  flat @click="deleteDialog = false" > Cancel </v-btn>
                    <v-spacer></v-spacer>
                    <v-btn color="error" @click="deletePilot" > Delete Pilot </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
          </v-flex>
        </v-layout>
      
    </div>

    <div v-else style="height: 90vh">
      <v-container style="height: 100%">
        <v-layout align-center justify-center row fill-height>
          <v-flex>
            <p class="grey--text text-xs-center display-2">NO PILOT LOADED</p>
          </v-flex>
        </v-layout>
      </v-container>
    </div>
  </div>
</template>

<script>
  import io from '@/store/data_io'
  import Stats from '@/logic/stats'
  import { EditableLabel, EditableTextfield } from '../UI'
  import { ImageSelector, BackgroundSelector, SkillSelector, TalentSelector, LicenseSelector, MechSkillsSelector, CoreBonusSelector } from './Selectors'
  import ContactsList from './ContactsList'
  import LicenseItem from './LicenseItem'
  import SkillItem from './SkillItem'
  import TalentItem from './TalentItem'
  import CoreBonusItem from './CoreBonusItem'
  import InvocationItem from './InvocationItem'
  import PilotLoadout from './LoadoutEditor/PilotLoadout'
  import NewConfig from '../HangarView/AddConfigMenu'

  export default {
    name: 'pilot-sheet',
    components: { EditableLabel, EditableTextfield, LicenseItem, SkillItem, TalentItem, PilotLoadout, CoreBonusItem, ImageSelector, ContactsList, BackgroundSelector, SkillSelector, TalentSelector, LicenseSelector, MechSkillsSelector, CoreBonusSelector, InvocationItem, NewConfig
    },
    data: () => ({
      callsignDialog: false,
      newCallsign: '',
      renameDialog: false,
      newName: '',
      invoke_trigger: '',
      invoke_attribute: null,
      newConfigModal: false,
      backgroundModal: false,
      appearanceModal: false,
      skillModal: false,
      licenseModal: false,
      talentModal: false,
      mechSkillModal: false,
      bonusModal: false,
      pilotGearModal: false,
      invokeDialog: false,
      deleteDialog: false,
      exportDialog: false,
      levelEditor: false,
      levelSkip: null,
      snackbar: false,
      notification: '',
      // loaders manage deletion and lazy loading of selectors
      newConfigLoader: false,
      appearanceLoader: false,
      skillLoader: false,
      licenseLoader: false,
      talentLoader: false,
      mechSkillLoader: false,
      cbLoader: false,
      contactKey: 0,
      activeLoadoutIdx: 0,
      loadoutForceReloadTrigger: 0
    }),
    methods: {
      refresh: function () {
        this.$forceUpdate()
      },
      setField: function (attr, val, close) {
        this[close] = false
        this.$store.dispatch('editPilot', {
          attr: attr,
          val: val
        })
      },
      item: function (type, id) {
        return this.$store.getters.getItemById(type, id)
      },
      getLicense: function (name) {
        return this.$store.getters.getLicenseByName(name.toLowerCase())
      },
      backgroundSelect: function (bgReturn) {
        this.backgroundModal = false
        this.$store.dispatch('editPilot', {
          attr: bgReturn.field,
          val: bgReturn.value
        })
      },
      setPilotSkills: function (skillArray) {
        this.skillModal = false
        this.skillLoader = false
        this.$store.dispatch('editPilot', {
          attr: `skills`,
          val: skillArray
        })
        this.$forceUpdate()
      },
      setPilotTalents: function (talentArray) {
        this.talentModal = false
        this.$store.dispatch('editPilot', {
          attr: `talents`,
          val: talentArray
        })
        this.$forceUpdate()
      },
      setPilotBonuses: function (bonusArray) {
        this.bonusModal = false
        this.$store.dispatch('editPilot', {
          attr: `core_bonuses`,
          val: bonusArray
        })
        this.$forceUpdate()
      },
      setLicenses: function (licenseArray) {
        this.licenseModal = false
        this.$store.dispatch('editPilot', {
          attr: `licenses`,
          val: licenseArray
        })
        this.$forceUpdate()
      },
      setPortrait: function (src) {
        this.$store.dispatch('editPilot', {
          attr: `portrait`,
          val: src
        })
        this.appearanceModal = false
        this.appearanceLoader = false
        this.notification = 'Pilot Portrait Saved'
        this.snackbar = true
      },
      setLevel: function () {
        if (this.levelSkip && this.levelSkip < this.pilot.level) {
          this.$store.dispatch('editPilot', {
            attr: `level`,
            val: parseInt(this.levelSkip)
          })
          this.$store.dispatch('editPilot', {
            attr: `licenses`,
            val: []
          })
          this.$store.dispatch('editPilot', {
            attr: `skills`,
            val: []
          })
          this.$store.dispatch('editPilot', {
            attr: `talents`,
            val: []
          })
          this.$store.dispatch('editPilot', {
            attr: `core_bonuses`,
            val: []
          })
          this.$store.dispatch('editPilot', {
            attr: `mechSkills`,
            val: {'hull': 0, 'agi': 0, 'sys': 0, 'eng': 0}
          })
        } else if (this.levelSkip) {
          this.$store.dispatch('editPilot', {
            attr: `level`,
            val: parseInt(this.levelSkip)
          })
        }
        this.levelEditor = false
        this.notification = `Pilot level changed to: ${this.levelSkip}`
        this.snackbar = true
      },
      addInvocation: function () {
        var inv = this.invoke_attribute === 0
          ? { accuracy: true }
          : { difficulty: true }
        inv.trigger = this.invoke_trigger

        var idx = this.pilot.invocations
          ? this.pilot.invocations.length
          : 0

        this.$store.dispatch('editPilot', {
          attr: `invocations[${idx}]`,
          val: inv
        })
        this.invokeDialog = false
        this.invoke_trigger = ''
        this.invoke_attribute = null
      },
      removeInvocation: function (tIndex) {
        this.$store.dispatch('splicePilot', {
          attr: 'invocations',
          start_index: tIndex,
          delete_count: 1
        })
      },
      goToConfig: function () {
        this.newConfigModal = false
        this.newConfigLoader = false
        this.$store.dispatch('loadPilot', this.pilot.id)
        this.$store.dispatch('loadConfig', this.pilot.configs[this.pilot.configs.length - 1])
        this.$router.push('/config')
      },
      deletePilot: function () {
        this.deleteDialog = false
        this.$store.dispatch('deletePilot', this.pilot.id)
      },
      clonePilot: function () {
        this.$store.dispatch('clonePilot', {id: this.pilot.id})
        this.notification = 'Pilot Duplicated'
        this.snackbar = true
      },
      exportPilot: function () {
        const { dialog } = require('electron').remote
        var path = dialog.showSaveDialog({
          defaultPath: this.pilot.callsign.toLowerCase().replace(/\W/g, ''),
          buttonLabel: 'Save Pilot'
        })
        io.saveFile(path + '.json', JSON.stringify(this.pilot), function (err) {
          if (err) {
            alert(`Error: COMP/CON could not save a file to ${path}`)
          } else {
            this.exportDialog = false
            this.notification = 'Pilot Export Successful'
            this.snackbar = true
          }
        })
      },
      copyPilot: function () {
        const {clipboard} = require('electron')
        clipboard.writeText(JSON.stringify(this.pilot))
        this.exportDialog = false
        this.notification = 'Pilot Data Copied to Clipboard'
        this.snackbar = true
      },
      openPrintOptions: function () {
        this.$store.dispatch('setPrintOptions', {loadout_index: this.activeLoadoutIdx})
        this.$router.push('/print-pilot')
      }
    },
    computed: {
      pilot: function () {
        return this.$store.getters.getPilot
      },
      stats: function () {
        if (this.loadoutForceReloadTrigger) console.info('Equipment changed: recalculating pilot stats...')
        else console.info('Loadout changed: recalculating pilot stats...')
        return Stats.pilotStats(this.pilot, this.pilot.loadouts[this.activeLoadoutIdx], this.$store.getters.getState)
      }
    },
    watch: {
      levelSkip: function () {
        if (this.levelSkip < 0) this.levelSkip = 0
        if (this.levelSkip > 12) this.levelSkip = 12
      }
    }
  }
</script>

<style scoped>
  .header {
    background-color: lightgray;
    color: black;
    font-weight: bold;
    letter-spacing: 3px;
    width: 100%;
    padding-left: 10px;
    margin-top:10px;
    margin-bottom: 3px;
  }

  .no-icon {
    height:40px;
    padding-top:8px
  }

  .v-dialog__activator {
    margin-left: -18px;
  }
  </style>

<style>
  .edit-btn {
  position: relative;
  /* margin-left: -10px!important; */
  opacity: 0.15;
  cursor: pointer;
  transition: 0.3s all;
}

  .mlneg {
    margin-left: -10px!important;
  }

  .edit-btn:hover {
    opacity: 1;
    transition: 0.3s all;
  }
</style>