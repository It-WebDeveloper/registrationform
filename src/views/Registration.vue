<template>
	<v-container fluid>
		<v-layout column>
			<!-- Название события -->
			<v-flex xs-12 v-if="eventInfo">
				<h1 class="text-lg-center text-md-center text-sm-center text-xs-center">
					{{eventInfo.eventTitle}}</h1>
			</v-flex>
			<v-flex xs-12>
				<v-layout my-1 row>
					<!-- Регистрация -->
					<registration-form v-if="goRegistration && eventInfo && personInfoComp !== undefined" 
													:event-info="eventInfo"
													:person-info="personInfoComp"
													:idparams="params"></registration-form>
					<!-- сообщение/ сообщение об ошибке -->
					<message v-if="showMessage" :titlemessage="message"></message>
				</v-layout>
			</v-flex>
		</v-layout>
	</v-container>
</template>

<script>

import axios from 'axios'
import RegistrationForm from '../components/RegistrationForm'
import Message from '../components/Message'
import {testGuid} from '../helpers/chekcer.js'

export default {
	name: 'registration',
	components: {
		RegistrationForm,
		Message
	},
	data() {
		return {
			params: {
				eventid: null,
				personid: null
			},
			lang: 'ru',
			error: null,
			errorMessage: null,
			goRegistration: false,
			showMessage: false,
			message: '',
			eventInfo: null,
			personInfo: undefined,
			isRegistered: false,
			skipRegistrationCheck: false
		}
	},
	created() {
		this.params.eventid = this.$route.params.eventGuid
		this.params.personid = this.$route.params.personGuid
		this.lang = window.myConfig.lang

		var paramsCheckResult = this.checkParams(this.params)
		
		this.showMessage = true
		switch (paramsCheckResult) {
			case 'noparams':
				this.message = this.$t('message.registrationPage.linkHasNoParams')
				// this.message = 'В ссылке нет параметров!'
				break;
			case 'parameternotcorrect':
				// this.message = 'Параметр в ссылке некорректен!'
				this.message = this.$t('message.registrationPage.paramIsWrong')
				break;
			case 'bothparamsnotcorrect':
				//this.message = 'Параметры в ссылке некорректны!'
				this.message = this.$t('message.registrationPage.paramsAreWrong')
				break;
			case 'secondparamnotcorrect':
				//this.message = 'Второй параметр в ссылке не корректен!'
				this.message = this.$t('message.registrationPage.secondParamIsWrong')
				break;
			case 'firstparamnotcorrect':
				//this.message = 'Первый параметр в ссылке не корректен!'
				this.message = this.$t('message.registrationPage.firstParamIsWrong')
				break;
			case 'nosecondparams':
				this.showMessage = false
				this.goRegistration = true
				this.skipRegistrationCheck = true
				this.personInfo = null
				break;
			case 'ok':
				this.showMessage = false
				this.goRegistration = true
				break;
			default: break;
		}
		if(!this.showMessage) {
			this.getEventInfo(this.params.eventid, this.lang)
		}
	},

	mounted() {
	},
	methods: {
		// Проверить параметры строки запроса
		checkParams(params) {
			// Наличие параметров и корректность параметров
			var eventidIsCorrect = false
			var personidIsCorrect  = false
			if (!params.eventid && !params.personid) {
				return 'noparams'
			}
			if (params.eventid && !params.personid) {
				eventidIsCorrect = testGuid(params.eventid)
				if (eventidIsCorrect) {
					return 'nosecondparams'
				} else {
					return 'parameternotcorrect' 
				}
			}
			
			eventidIsCorrect = testGuid(params.eventid)
			personidIsCorrect = testGuid(params.personid)
			
			if (!eventidIsCorrect && !personidIsCorrect) {
				return 'bothparamsnotcorrect'
			}
			if (!eventidIsCorrect && personidIsCorrect) {
				return 'firstparamnotcorrect'
			}
			if (eventidIsCorrect && !personidIsCorrect) {
				return 'secondparamnotcorrect'
			}
			return 'ok'
		},
		// Получить информция о событии
		getEventInfo(eventid, lang) {
			axios.post(window.myConfig.WsUrl, {
				calcId: window.myConfig.ws.getEventInfo,
				args: JSON.stringify({ eventGuid: eventid, lang}),
				ticket: ''
			})
			.then(response => {
				this.eventInfo = JSON.parse(response.data.d)
				if (this.eventInfo) {
					this.goRegistration = true
					this.showMessage = false
					if (!this.skipRegistrationCheck) {
						this.hasRegistered(this.params.eventid, this.params.personid, this.lang)
					}
				} else {
					this.goRegistration = false
					this.showMessage = true
					//this.message = 'Ссылка не связана ни с одним из событий!'
					this.message = this.$t('message.registrationPage.linkHasNotCorrespondin')
				}
				})
			.catch(error => {
				this.error = error
				this.goRegistration = false
				this.showMessage = true
				if (error.response) {
						// ответ получен, но ошибка
						this.message = this.$t('message.registrationPage.responseButError', {'errorresponsestatus': error.response.status, 'errorresponsedata': error.response.data})
					} else if (error.request) {
						//this.message = 'Ответ от сервера не получен!'
						this.message = this.$t('message.registrationPage.serverNotResponse')
					} else {
						this.message = error.message
					}
			})
		},

		// Уже есть регистрация
		hasRegistered ( eventid, personid, lang ) {
			axios.default.withCredentials = true;
			axios.post(window.myConfig.WsUrl, {
				calcId: window.myConfig.ws.hasRegistration,
				args: JSON.stringify({ eventGuid: eventid, personGuid: personid, lang}),
				ticket: ''
			})
			.then( response => {
				this.personInfo = JSON.parse(response.data.d)
				this.isRegistered = this.personInfo.isRegistered
				if(this.isRegistered) {
					this.goRegistration = false
					this.showMessage = true
					this.message = this.$t('message.registrationPage.youHaveRegistration')
					this.personInfo = null
				} else {
					this.goRegistration = true
					this.showMessage = false
				}
			})
			.catch(error => {
				this.error = error
				this.goRegistration = false
				this.showMessage = true
				if (error.response) {
						// ответ получен, но ошибка
						this.message = this.$t('message.registrationPage.responseButError', {'errorresponsestatus': error.response.status, 'errorresponsedata': error.response.data})
					} else if (error.request) {
						// запрос выполнен, но ответ не получен
						this.message = this.$t('message.registrationPage.serverNotResponse')
					} else {
						this.message = error.message
					}
			})
		},
	},
	computed: {
		eventInfoData() {
			return this.eventInfo
		},
		goToRegistration() {
			return this.goRegistration
		},
		personInfoComp() {
			return this.personInfo
		}
	}
}
</script>

<style scoped>
.light-gray-color {
	background: #fafafa
}
</style>