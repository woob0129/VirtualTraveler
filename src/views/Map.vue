<template>
	<div class="map-wrapper">
		<div class="map" id="mapContainer"></div>
		<ChatRoomInfo />
		
		<v-dialog v-model="joinDialog" max-width="390" >
			<v-card>
			<v-card-title class="mb-5">🎈 채팅방에 참여하시겠습니까?</v-card-title>
			<v-card-actions>
				<v-spacer></v-spacer>
				<v-btn
					color="primary darken-1"
					text
					@click="join()"
				>
					Yes
				</v-btn>
				<v-btn
					color="warning"
					text
					@click="joinDialog = false"
				>
					No
				</v-btn>
			</v-card-actions>
			</v-card>
		</v-dialog>
		<v-dialog v-model="createDialog" persistent max-width="390px">
			<template v-slot:activator="{ on }">
			<v-btn color="primary" dark v-on="on">Open Dialog</v-btn>
			</template>
			<v-card>
			<v-card-title>🚀 채팅방 이름을 입력하세요.</v-card-title>
			<v-card-text>
				<v-container>
					<v-row>
						<v-col cols="12">
							<v-text-field v-model="title" label="Title" required />
						</v-col>
					</v-row>
				</v-container>
			</v-card-text>
			<v-card-actions>
				<v-spacer></v-spacer>
				<v-btn color="red darken-1" text @click="createDialog = false">Cancel</v-btn>
				<v-btn color="blue darken-1" text @click="createChatRoom()">Create</v-btn>
			</v-card-actions>
			</v-card>
		</v-dialog>
		<input class="searchFrom" v-model="searchdata" placeholder="검색" @keydown.enter="search()"/>
		<img
			v-if="!isPresent"
			src="@/images/location.png"
			class="marker"
			@click="create()"
		/>
		<img
			v-if="isPresent"
			src="@/images/cancel.png"
			class="marker"
			@click="cancel()"
		/>
		<img
			src="@/images/unlocked.png"
			class="logout"
			@click="logout()"
		/>
		{{this.setAddress}}
	</div>
</template>

<script>

import ChatRoomInfo from '@/components/ChatRoomInfo.vue';
import firebaseApi from '@/api/firebaseApi.js';
import kakaomapApi from '@/api/kakaomapApi.js';
import { mapGetters, mapState } from "vuex";
import { HashMap } from '../utils/hashMap.js';
import storage from '../utils/storage.js';

export default {
    components: {
		ChatRoomInfo
    },
    data() {
        return {
            map: '',
			currentLocation: '',
			isPresent: false,
			joinDialog: false,
			createDialog: false,
			marker: null,
			title: '',
			location: null,
			unsubscribe: null,
			viewPoint: null,
			searchdata : '',
			markers: []
		}
	},
    computed: {
		...mapGetters(['getChatRooms', 'getMarkers', 'getLoginUser', 'getOnlineChatRoom']),
		setAddress : function(){
			const _this = this;
			if(this.getOnlineChatRoom==null) return;
			const result = kakaomapApi.getAddress(this.getOnlineChatRoom.location);
			result.then(data=>{
				var geocoder = new kakao.maps.services.Geocoder();
				geocoder.addressSearch(data, function(result, status) {
					if (status === kakao.maps.services.Status.OK) {
						var bounds = new kakao.maps.LatLngBounds();
						bounds.extend(new kakao.maps.LatLng(result[0].y, result[0].x));
						_this.map.setBounds(bounds);
					} 
				});    
			});
		},
	},
	mounted() {
		if (navigator.geolocation) {
			navigator.geolocation.getCurrentPosition(position => { this.init(position); }, this.failToGetGeoInfo);
		} else {
			console.log('지형정보를 지원하지 않는 환경입니다.');
		}
		this.$store.watch(() => { this.getMarkers.values() }, () => { this.updateMarker(); }, {deep: true});
	},
	beforeDestroy() {
		if(this.unsubscribe!==null){
			this.unsubscribe();
		}
	},
    methods: {
		search(){
			const _this = this;
			var ps = new kakao.maps.services.Places(); 
			ps.keywordSearch(this.searchdata, (data, status, pagination) =>{
				if (status === kakao.maps.services.Status.OK) {
					var bounds = new kakao.maps.LatLngBounds();
					bounds.extend(new kakao.maps.LatLng(data[0].y, data[0].x));
					_this.map.setBounds(bounds);
					_this.searchdata ='';
				} else{ //검색이 안됐을 때
					_this.searchdata ='';
				}
			}); 
		},
		async init(position) {
			const mapContainer = document.getElementById("mapContainer");
			this.map = await kakaomapApi.drawMap(mapContainer, position);
			this.unsubscribe = await this.$store.dispatch('fetchChatRooms');
        },
        failToGetGeoInfo() {
            // 위치 정보 수집 실패 핸들링
            alert("위치 정보를 받아오는데 실패하였습니다.");
		},
        async create() {
			this.isPresent = true;
			this.map.addOverlayMapTypeId(kakao.maps.MapTypeId.ROADVIEW);
            const center = this.map.getCenter();
			this.marker = await kakaomapApi.createSelectionMarker(center);
			const _this = this;
			kakao.maps.event.addListener(this.marker, 'click', function() {
				const rvClient = new kakao.maps.RoadviewClient();
				rvClient.getNearestPanoId(_this.marker.getPosition(), 50, function(panoId){
					if(panoId===null){
						alert('🛸 로드뷰가 지원되지 않는 지역입니다.')
					} else {
						_this.location = {
							latitude: _this.marker.getPosition().Ha,
							longitude: _this.marker.getPosition().Ga
						}
						_this.viewPoint = {
							pan: 30,
							tilt: -10,
							zoom: 1
						}
						_this.createDialog = true;
					}
				});
			});
			this.marker.setMap(this.map);
		},
		cancel() {
			this.isPresent = false;
			this.map.removeOverlayMapTypeId(kakao.maps.MapTypeId.ROADVIEW);
			this.marker.setMap(null);
		},
		updateMarker() {
			this.removeMarker();
			this.drawMarker();
		},
		removeMarker() {
			this.markers.forEach((marker)=>{
				marker.setMap(null);
			});
			this.markers = [];
		},
        drawMarker() {
			this.getMarkers.values().forEach((marker) => {
				marker.setMap(this.map);
				this.markers.push(marker);
				kakao.maps.event.addListener(marker, 'click', () => {
					this.$store.commit('updateSelectedId', marker.getTitle());
					this.joinDialog = true;
				});
			})
		},
		async createChatRoom() {
			if(this.title===''||this.title===null){
				alert('☠ 채팅방 이름을 입력하세요.');
			}else{
				const room = {
					title: this.title,
					location: this.location,
					viewPoint: this.viewPoint
				};
				await this.$store.dispatch('createChatRoom', room);
				storage.join(this.getOnlineChatRoom);
				this.$router.push('/travel');
			}
		},
		async join() {
			await firebaseApi.joinChatRoom(this.getOnlineChatRoom, this.getLoginUser);
			storage.join(this.getOnlineChatRoom);
			this.$router.push('/travel');
		},
        async logout() {
            await this.$store.dispatch('logout');
            this.$router.push('/auth/login');
        }
    }
}
</script>

<style lang="scss">
.map {
	height: 100vh;
	width: 100vw;
}
.searchFrom{
	left: 400px;
    top: 22px;
	position: fixed;
	height: 40px;
	width: 300px;
	border-radius: 20px;
	z-index: 9999;
	background-color: white;
	opacity: 0.9;
	padding-left: 10px;
}
.searchFrom:focus {
	outline: none;
}
.marker {
	left: 325px;
	top: 10px;
	position: fixed;
	height: 64px;
	width: 64px;
	border-radius: 50px;
	cursor: pointer;
	z-index: 9999;

	-webkit-transition: all 0.3s ease-in-out;
	-moz-transition: all 0.3s ease-in-out;
	-o-transition: all 0.3s ease-in-out;
	-ms-transition: all 0.3s ease-in-out;
	&:hover {
		opacity: 0.7;
		cursor: pointer;
	}
}

.logout {
	right: 10px;
	top: 10px;
	position: fixed;
	height: 64px;
	width: 64px;
	border-radius: 50px;
	cursor: pointer;
	z-index: 9999;

	-webkit-transition: all 0.3s ease-in-out;
	-moz-transition: all 0.3s ease-in-out;
	-o-transition: all 0.3s ease-in-out;
	-ms-transition: all 0.3s ease-in-out;
	&:hover {
		opacity: 0.7;
		cursor: pointer;
	}
}
</style>