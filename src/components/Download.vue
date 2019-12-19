<template>
	<v-content>
		<message v-if="messageTitle" :titlemessage="messageTitle" :message="messageContent"></message>
	</v-content>
</template>

<script>
import axios from 'axios'
import Message from './Message'
import {testGuid} from '../helpers/chekcer.js'

export default {
	name: 'download',
	components: {Message},
	data() {
		return {
			eventGuid: '',
			personGuid: '',
			link: '',
			messageTitle: '',
			messageContent: ''
		}
	},
	created() {
		document.title = 'Загрузка файла'
		// получить параметры строки запроса
		this.eventGuid = this.$route.params.eventGuid
		this.personGuid = this.$route.params.personGuid

		// выполнить проверку параметров
		var checkParams = this.checkParams(this.eventGuid, this.personGuid)

		switch (checkParams) {
			case 'noparams':
				this.messageTitle = 'Ошибка'
				this.messageContent = 'В ссылке нет всех параметров!'
				break;
			case 'firstParamWrong':
				this.messageTitle = 'Ошибка'
				this.messageContent = '1-й параметр некорректен!'
				break;
			case 'secondParamWrong':
				this.messageTitle = 'Ошибка'
				this.messageContent = '2-й параметр некорректен!'
				break;
			case 'ok':
				// получить ссылку на файл
				this.getLinkForDownload(this.eventGuid, this.personGuid)
				break;
		}
	},
	mounted() {
	},

	methods: {
		checkParams: function( eventGuid, personGuid ) {
			// проверка наличия
			if (!(eventGuid && personGuid)) {
				return 'noparams'
			}
			// проверка корректности
			var eventGuidIsCorrect = testGuid(eventGuid)
			var personGuidIdCorrect = testGuid(eventGuid)
			if (eventGuidIsCorrect && personGuidIdCorrect) {
				return 'ok'
			}
			if (!eventGuidIsCorrect) {
				return 'firstParamWrong'
			} else {
				return 'secondParamWrong'
			}

		},
		getLinkForDownload( eventGuid, personGuid ) {
			axios.post(window.myConfig.WsUrl, {
				calcId: window.myConfig.ws.getFile,
				args: JSON.stringify({eventGuid, personGuid}),
				ticket: ''
			})
			.then(response => {
				var resp = JSON.parse(response.data.d)
				if (resp.success) {
					if(resp.link) {
						window.location = resp.link
					} else {
						// link unavailable
						this.messageTitle = 'Ссылка сейчас недоступна!'
					}
				} else {
					this.messageTitle = resp.message
				}
			})
			.catch(error => {
				if (error.response) {
					this.messageContent = 'Status: ' + error.response.status + '<br>' + error.response.header
				} else if (error.request) {
					this.messageContent = error.request
				} else {
					this.messageContent = error.message
				}
				this.messageTitle = 'Ошибка!'
			})
		}
	}
}
</script>

<style>

</style>