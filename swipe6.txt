// ==UserScript==
// @name         Swipe Navigation
// @namespace    http://tampermonkey.net/
// @version      0.5
// @description  1손가락 스와이프로 페이지 탐색하기 (상단 45% 제외, 좌우 30% 조건 추가)
// @match        *://*/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';

    let startX, startY, startTime, isTouching = false;
    let backTimer;

    // 전체화면 여부 확인
    function isFullScreen() {
        return document.fullscreenElement || document.webkitFullscreenElement || document.mozFullScreenElement;
    }

    function isVideoPlayer(target) {
        while (target) {
            if (target.tagName === 'VIDEO' || 
                (target.className && target.className.toLowerCase().includes('video')) || 
                (target.className && target.className.toLowerCase().includes('player')) || 
                (target.dataset && Object.keys(target.dataset).some(key => key.toLowerCase().includes('video') || key.toLowerCase().includes('player')))) {
                return true;
            }
            target = target.parentElement;
        }
        return false;
    }

    function handleTouchStart(event) {
        if (event.touches.length !== 1) return;  // 터치가 1개가 아니면 리턴

        let target = event.target;

        // 유튜브 사이트에서 전체화면이 아닐 때는 html에 터치 이벤트 처리
        if (window.location.hostname.includes('youtube.com')) {
            if (!isFullScreen()) {
                let html = document.documentElement;  // <html> 요소
                if (target === html) return;  // 전체화면이 아닐 때 html 터치 시 이벤트 처리
            }
            if (isFullScreen()) {
                if (isVideoPlayer(target)) return;  // 전체화면일 때 비디오 플레이어를 터치하면 리턴
            }
        }

        // 유튜브 사이트가 아니거나 전체화면이 아닐 때는 기존 로직
        if (isFullScreen() && isVideoPlayer(target)) return;  // 기존 로직




        startX = event.touches[0].clientX;
        startY = event.touches[0].clientY;

        if (startY < (window.innerHeight * 0.45)) return;
        if (startX > (window.innerWidth * 0.3) && startX < (window.innerWidth * 0.7)) return;

        startTime = new Date().getTime();
        isTouching = true;
    }

    function handleTouchMove(event) {
        if (event.touches.length !== 1) {
            isTouching = false;
            return;
        }
        if (!isTouching) return;

        let currentX = event.touches[0].clientX;
        let currentY = event.touches[0].clientY;

        // 무브 시점에서도 가로 30% 제한 적용
        if (currentX > (window.innerWidth * 0.3) && currentX < (window.innerWidth * 0.7)) {
            isTouching = false;
            return;
        }

        let deltaX = currentX - startX;
        let deltaY = currentY - startY;

        if (Math.abs(deltaY) > Math.abs(deltaX)) {
            isTouching = false;
        }
    }

    function handleTouchEnd(event) {
        if (event.changedTouches.length !== 1) return;
        if (!isTouching) return;

        let endX = event.changedTouches[0].clientX;
        let endY = event.changedTouches[0].clientY;

        // 엔드 시점에서도 가로 30% 제한 적용
        if (endX > (window.innerWidth * 0.3) && endX < (window.innerWidth * 0.7)) {
            isTouching = false;
            return;
        }

        let deltaX = endX - startX;
        let deltaY = endY - startY;
        let deltaTime = new Date().getTime() - startTime;

        if (Math.abs(deltaX) > 2 * Math.abs(deltaY) && deltaTime < 300) {
            if (deltaX > 75) {
                window.history.forward();
            } else if (deltaX < -75) {
                window.history.back();

                backTimer = setTimeout(() => {
                    window.close();
                }, 200);
            }
        }

        isTouching = false;
    }

    window.addEventListener('popstate', () => {
        clearTimeout(backTimer);
    });

    document.addEventListener('touchstart', handleTouchStart, { passive: true });
    document.addEventListener('touchmove', handleTouchMove, { passive: true });
    document.addEventListener('touchend', handleTouchEnd);
})();
