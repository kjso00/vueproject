<template>
    <div class="container">
        <div class="total">
            <header class="navigation">
                <div class="first">
                    <div class="logo">
                        음식점
                    </div>
                    <div class="gps">
                        <!-- 위치 표시 -->
                        <div class="gpslocation">{{ location }}</div> 
                        <div>
                            <img src="https://img.icons8.com/?size=100&id=3723&format=png&color=000000" alt="gps">
                        </div>
                    </div>
                </div>
                <div class="second">
                    <div class="search">
                        <input type="text" placeholder="위치를 입력하세요" v-model="keyword">
                        <button class="button" @click="handleSearch">검색</button>
                    </div>
                </div>
            </header>

            <div class="totalbody">
                <!-- 장소 정보를 표시할 div -->
                <div ref="map" class="map">
                    <!-- 마커 정보 리스트 -->
                    <div class="placeinfo" v-if="selectedPlace">
                        <div class="title">
                            <div>
                                <strong>{{ selectedPlace.place_name }}</strong>
                            </div>
                            <div class="bookcall">
                                <div>
                                    <button class="booking" @click="addToFavorites(selectedPlace)">
                                        <img src="https://icons.iconarchive.com/icons/github/octicons/48/bookmark-24-icon.png"
                                            alt="북마크">
                                    </button>
                                </div>
                                <div class="call" @click="togglePopup2">
                                    <button>
                                        <img src="https://icons.iconarchive.com/icons/iynque/ios7-style/48/Phone-icon.png"
                                            alt="전화">
                                    </button>
                                </div>
                            </div>
                        </div>
                        <div class="innerinfo">
                            <p>{{ selectedPlace.address_name }}</p>
                            <div class="number">
                                <p>{{ selectedPlace.phone }}</p>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 전화 팝업 -->
                <div v-if="showPopup2" class="popup2">
                    <div class="popup-content2">
                        <span class="close" @click="togglePopup2">&times;</span> <!-- 닫기 버튼 -->
                        <p>{{ selectedPlace.phone }}
                        <div class="buttoneffect1">
                            <img src="https://icons.iconarchive.com/icons/praveen/minimal-outline/72/back-2-icon.png"
                                alt="지우기">
                        </div>
                        </p> <!-- 전화번호 표시 -->
                        <div class="keypad">
                            <div class="keypad-row">
                                <button @click="addNumber(1)">1</button>
                                <button @click="addNumber(2)">2</button>
                                <button @click="addNumber(3)">3</button>
                            </div>
                            <div class="keypad-row">
                                <button @click="addNumber(4)">4</button>
                                <button @click="addNumber(5)">5</button>
                                <button @click="addNumber(6)">6</button>
                            </div>
                            <div class="keypad-row">
                                <button @click="addNumber(7)">7</button>
                                <button @click="addNumber(8)">8</button>
                                <button @click="addNumber(9)">9</button>
                            </div>
                            <div class="keypad-row">
                                <button @click="addNumber('*')">*</button>
                                <button @click="addNumber(0)">0</button>
                                <button @click="addNumber('#')">#</button>
                            </div>
                            <div class="keypad-row">
                                <div class="buttoneffect">
                                    <img src="https://icons.iconarchive.com/icons/iynque/ios7-style/72/Phone-icon.png"
                                        alt="전화">
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <footer>
                <nav class="bottom">
                    <div class="bookmark" @click="togglePopup">
                        <a href="#">
                            <RouterLink to="/main/favorites">
                            <img src="https://img.icons8.com/ios/50/000000/bookmark-ribbon--v1.png" alt="즐겨찾기"
                                style="width: 35px; height: 35px;">
                            </RouterLink>
                            </a>
                    </div>
                    <div class="home">
                        <RouterLink to="/main">
                            <img src="https://img.icons8.com/ios/50/000000/home-page.png" alt="홈">
                        </RouterLink>
                    </div>
                    <div class="logout">
                        <RouterLink to="/">
                            <img src="https://img.icons8.com/ios/50/000000/logout-rounded-left.png" alt="로그아웃">
                        </RouterLink>
                    </div>
                </nav>
            </footer>

            <!-- 팝업창 -->
            <div v-if="showPopup" class="popup">
                <div class="popup-content">
                    <span class="close" @click="togglePopup">&times;</span>
                    <p>
                        <즐겨찾기>
                    </p>
                    <div class="poptitle">
                        <strong>{{ selectedPlace.place_name }}</strong>
                    </div>
                    <p>{{ selectedPlace.address_name }}</p>
                    <div class="number">
                        <p>{{ selectedPlace.phone }}</p>
                    </div>
                </div>
            </div>

        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'; // Vue에서 ref와 onMounted 함수를 가져옵니다.
import "../css/style.css";
import store from '../store';


const currentLocation = ref(''); // 위치를 표시할 변수
const location = ref(''); // 위치를 표시할 변수
// 검색어를 저장할 변수
const keyword = ref('');
// 장소 목록을 저장할 변수
const places = ref([]);
// 지도 인스턴스를 저장할 변수
let mapInstance;
// 장소 검색 서비스를 저장할 변수
const ps = ref(null);
// 지도 경계 변경 이벤트 리스너를 저장할 변수
const boundsChangedListener = ref(null);

// 선택된 장소 정보를 저장할 변수
const selectedPlace = ref(null);

// 팝업창 상태를 저장할 변수
const showPopup = ref(false);
const showPopup2 = ref(false);

const phoneNumber = ref(''); // 전화번호를 저장할 변수

// 팝업창 열기/닫기 함수
const togglePopup = () => {
    showPopup.value = !showPopup.value;
};

// 전화번호에 숫자를 추가하는 함수
const addNumber = (number) => {
    phoneNumber.value += number;
};

// 전화번호를 초기화하는 함수
const clearPhoneNumber = () => {
    phoneNumber.value = '';
};

// 새로운 팝업창 열기/닫기 함수
const togglePopup2 = () => {
    showPopup2.value = !showPopup2.value;
    clearPhoneNumber(); // 팝업이 열릴 때마다 전화번호를 초기화합니다.
};


// 컴포넌트가 마운트되었을 때 실행될 함수
onMounted(() => {
    loadKakaoMapScript();
});

const searchlocation = () => {
    location.value = keyword.value; // 검색어를 위치에 반영
    // 이후에 검색 결과를 처리하는 코드를 추가할 수 있습니다.
};

// 검색 함수

const search = async () => {
    // Kakao Maps API의 geocoder 서비스를 사용하여 검색어를 좌표로 변환
    const geocoder = new kakao.maps.services.Geocoder();

    geocoder.addressSearch(keyword.value, (result, status) => {
        if (status === kakao.maps.services.Status.OK) {
            const coords = new kakao.maps.LatLng(result[0].y, result[0].x);
            currentLocation.value = coords; // 변환된 좌표 저장
            moveMapTo(coords); // 지도 이동 함수 호출
        } else {
            alert('검색 결과가 없습니다.');
        }
    });
};

// 지도 이동 함수

const moveMapTo = (coords) => {
    mapInstance.setCenter(coords); // 지도 이동
};

// 카카오 맵 스크립트를 로드하는 함수
const loadKakaoMapScript = () => {
    const script = document.createElement('script'); // 새로운 스크립트 태그 생성
    script.src = 'https://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=9f647a199751b8c18f579fff34313544&libraries=services'; // 카카오 맵 스크립트 URL 설정
    script.onload = () => { // 스크립트가 로드되면 실행될 함수
        kakao.maps.load(() => { // 카카오 맵이 로드되면 실행될 함수
            initializeKakaoMap();
        });
    };
    document.head.appendChild(script); // 스크립트를 문서의 head에 추가
};

// 카카오 맵을 초기화하는 함수
const initializeKakaoMap = () => {
    const options = {
        center: new kakao.maps.LatLng(37.498851, 127.026181), // 초기 지도 중심 좌표
        level: 5 // 초기 지도 확대 레벨
    };
    const mapContainer = document.querySelector('.map'); // 지도를 표시할 div 요소 선택
    if (!mapContainer) return; // 만약 mapContainer가 없다면 함수 종료

    mapInstance = new kakao.maps.Map(mapContainer, options); // 새로운 지도 인스턴스 생성
    ps.value = new kakao.maps.services.Places(mapInstance); // 장소 검색 서비스 생성

    // 장소 검색 및 마커 표시
    searchPlaces();

    // 지도 경계 변경 이벤트 리스너 등록
    boundsChangedListener.value = kakao.maps.event.addListener(mapInstance, 'bounds_changed', handleMapDragEnd);
};

// 장소를 검색하는 함수
const searchPlaces = () => {
    if (!ps.value) return; // 장소 검색 서비스가 초기화되지 않았으면 함수 종료
    ps.value.categorySearch('FD6', placesSearchCB, { useMapBounds: true }); // 카테고리 코드 'FD6'을 사용해 장소 검색
};

// 장소 검색 콜백 함수
const placesSearchCB = (data, status) => {
    if (status === kakao.maps.services.Status.OK) { // 검색이 성공했을 때
        places.value = data; // 검색 결과를 places 변수에 저장
        displayPlaces(data); // 검색 결과를 지도에 표시
    } else {
        // 에러 처리
    }
};

// 장소를 지도에 표시하는 함수
const displayPlaces = (places) => {
    removeAllMarkers(); // 기존 마커 제거
    places.forEach(place => {
        displayMarker(place); // 각 장소에 마커 표시
    });
};

// 장소에 마커를 표시하는 함수
const displayMarker = (place) => {
    const marker = new kakao.maps.Marker({
        map: mapInstance, // 마커를 표시할 지도 인스턴스
        position: new kakao.maps.LatLng(place.y, place.x) // 마커의 위치 좌표
    });

    // 마커 클릭 시 장소 정보 표시
    kakao.maps.event.addListener(marker, 'click', () => {
        selectedPlace.value = place; // 선택된 장소 정보를 업데이트
    });

    marker.setMap(mapInstance); // 마커를 지도에 표시
};

// 지도 드래그 종료 시 실행될 함수
const handleMapDragEnd = () => {
    const bounds = mapInstance.getBounds(); // 현재 지도 경계를 가져옴
    const swLatLng = bounds.getSouthWest(); // 남서쪽 좌표
    const neLatLng = bounds.getNorthEast(); // 북동쪽 좌표

    ps.value.categorySearch('FD6', (data, status) => {
        if (status === kakao.maps.services.Status.OK) { // 검색이 성공했을 때
            displayPlaces(data); // 검색 결과를 지도에 표시
        } else {
            // 에러 처리
        }
    }, {
        bounds: new kakao.maps.LatLngBounds(swLatLng, neLatLng) // 현재 지도 경계를 기준으로 검색
    });
};

// 모든 마커를 제거하는 함수
const removeAllMarkers = () => {
    markers.forEach(marker => {
        marker.setMap(null); // 마커를 지도에서 제거
    });
    markers = []; // 마커 배열 초기화
};

// 마커 배열
let markers = [];

const showAlert = () => {
    alert("즐겨찾기가 등록되었습니다.");
    addToFavorites(selectedPlace.value);
};

const placeData = ref({
    place_name: '',
    address_name: '',
    phone: ''
});

// 마커 클릭 시 해당 장소 정보를 팝업에 표시하는 함수
const showPlaceInfo = (place) => {
    placeData.value = {
        place_name: place.place_name,
        address_name: place.address_name,
        phone: place.phone
    };
    togglePopup(); // 팝업 창 열기
};

// JSON 형식으로 즐겨찾기에 추가된 장소 정보를 저장할 변수
const favoritePlacesData = ref([]);

// 즐겨찾기 버튼을 클릭했을 때 실행될 함수
const addToFavorites = (place) => {
    alert("즐겨찾기가 등록되었습니다.");
    store.addToFavorites(place); // 전역 상태에 추가
    // togglePopup(); // 팝업 창 닫기
};

// 즐겨찾기 페이지로 이동하는 함수
const gotoFavoritesPage = () => {
    // Vue Router를 사용하여 즐겨찾기 페이지로 이동
};

// 즐겨찾기 목록에서 장소를 제거하는 함수
const removeFromFavorites = (index) => {
    store.removeFromFavorites(index); // 전역 상태에서 제거
};

const handleSearch = () => {
    search();
    searchlocation();
};

</script>

<style scoped></style>