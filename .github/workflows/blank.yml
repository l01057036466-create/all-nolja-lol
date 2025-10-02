<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>롤 솔랭 점수 계산기</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
        }
        .score-display {
            font-size: 2rem;
            font-weight: 700;
        }
        .btn {
            transition: all 0.2s ease-in-out;
        }
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }
        .btn:active {
            transform: translateY(0);
            box-shadow: none;
        }
        .game-log::-webkit-scrollbar {
            width: 4px;
        }
        .game-log::-webkit-scrollbar-thumb {
            background-color: #4A5568;
            border-radius: 4px;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3498db;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        #loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.7);
            z-index: 9999;
            display: flex;
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body class="bg-gray-900 text-white min-h-screen p-4 sm:p-6 md:p-8">
    <div id="loading-overlay">
        <div class="loader"></div>
    </div>

    <div class="max-w-6xl mx-auto">
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-bold text-cyan-400">명절치킨배 솔랭 점수내기</h1>
            <p class="text-gray-400 mt-2">참가자별 점수를 실시간으로 관리하세요.</p>
        </header>

        <div class="bg-gray-800 p-4 rounded-xl shadow-lg mb-8 text-center">
             <p class="text-sm text-gray-400">나의 접속 ID (공유 금지)</p>
             <p id="user-id" class="text-cyan-300 font-mono text-xs break-all">연결 중...</p>
             <p id="admin-status" class="text-yellow-400 font-bold mt-2 hidden">👑 관리자</p>
        </div>

        <div id="admin-panel" class="bg-gray-800 p-6 rounded-xl shadow-lg mb-8 hidden">
            <h2 class="text-2xl font-semibold mb-4 text-white">참가자 추가 (관리자)</h2>
            <form id="add-player-form" class="flex flex-col sm:flex-row gap-4">
                <input type="text" id="player-name" placeholder="닉네임" class="flex-grow bg-gray-700 text-white placeholder-gray-400 rounded-md px-4 py-2 border border-gray-600 focus:ring-2 focus:ring-cyan-500 focus:outline-none" required>
                <input type="text" id="player-tier" placeholder="현재 티어" class="flex-grow bg-gray-700 text-white placeholder-gray-400 rounded-md px-4 py-2 border border-gray-600 focus:ring-2 focus:ring-cyan-500 focus:outline-none" required>
                <select id="player-team" class="bg-gray-700 text-white rounded-md px-4 py-2 border border-gray-600 focus:ring-2 focus:ring-cyan-500 focus:outline-none">
                    <option value="1">1팀</option>
                    <option value="2">2팀</option>
                </select>
                <button type="submit" class="btn bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-2 px-6 rounded-md shadow-md">추가</button>
            </form>
        </div>

        <div class="mb-8">
            <h2 class="text-2xl font-semibold mb-4 text-white">팀 점수 합산</h2>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-6">
                <div class="bg-blue-900/50 p-6 rounded-xl shadow-lg text-center border border-blue-700">
                    <h3 class="text-xl font-bold text-blue-300 mb-2">1팀 총점</h3>
                    <p id="team1-score" class="score-display text-white">0</p>
                </div>
                <div class="bg-red-900/50 p-6 rounded-xl shadow-lg text-center border border-red-700">
                    <h3 class="text-xl font-bold text-red-300 mb-2">2팀 총점</h3>
                    <p id="team2-score" class="score-display text-white">0</p>
                </div>
            </div>
        </div>

        <div>
            <h2 class="text-2xl font-semibold mb-4 text-white">참가자 목록</h2>
            <div id="player-list" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
            </div>
             <div id="no-players-message" class="text-center py-10 text-gray-500">
                <p>아직 추가된 참가자가 없습니다. (관리자가 추가할 수 있습니다)</p>
            </div>
        </div>
        
        <div class="mt-12">
            <h2 class="text-2xl font-semibold mb-4 text-white">참가자별 전적 스크린샷</h2>
            <div id="screenshot-list" class="bg-gray-800 p-6 rounded-xl shadow-lg">
                <p id="no-screenshots-message" class="text-gray-500">아직 제출된 스크린샷이 없습니다.</p>
            </div>
        </div>

    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, addDoc, updateDoc, deleteDoc, onSnapshot, collection, serverTimestamp, arrayUnion } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const addPlayerForm = document.getElementById('add-player-form');
        const playerNameInput = document.getElementById('player-name');
        const playerTierInput = document.getElementById('player-tier');
        const playerTeamSelect = document.getElementById('player-team');
        const playerList = document.getElementById('player-list');
        const noPlayersMessage = document.getElementById('no-players-message');
        const team1ScoreEl = document.getElementById('team1-score');
        const team2ScoreEl = document.getElementById('team2-score');
        const loadingOverlay = document.getElementById('loading-overlay');
        const adminPanel = document.getElementById('admin-panel');
        const userIdEl = document.getElementById('user-id');
        const adminStatusEl = document.getElementById('admin-status');
        const screenshotListEl = document.getElementById('screenshot-list');
        const noScreenshotsMessage = document.getElementById('no-screenshots-message');

        const SCORE_RULES = { WIN: 20, LOSS: -20, SNIPE_WIN: 25 };
        
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'lol-score-tracker-default';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        
        if (!firebaseConfig) {
            alert("Firebase 설정이 로드되지 않았습니다. 앱을 다시 로드해주세요.");
            loadingOverlay.style.display = 'none';
        }

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let currentUser = null;
        let isAdmin = false;

        onAuthStateChanged(auth, async (user) => {
            if (user) {
                currentUser = user;
                userIdEl.textContent = currentUser.uid;
                await checkAdminAndLoadData();
            } else {
                try {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (error) {
                    console.error("Authentication Error:", error);
                    alert("인증에 실패했습니다. 페이지를 새로고침 해주세요.");
                }
            }
        });

        async function checkAdminAndLoadData() {
            const eventRef = doc(db, `artifacts/${appId}/public/data/eventConfig/config`);
            const eventSnap = await getDoc(eventRef);

            if (!eventSnap.exists()) {
                await setDoc(eventRef, { adminId: currentUser.uid, createdAt: serverTimestamp() });
                isAdmin = true;
            } else {
                isAdmin = eventSnap.data().adminId === currentUser.uid;
            }
            
            adminStatusEl.style.display = isAdmin ? 'block' : 'none';
            adminPanel.style.display = isAdmin ? 'block' : 'none';
            
            const playersRef = collection(db, `artifacts/${appId}/public/data/players`);
            onSnapshot(playersRef, (querySnapshot) => {
                const players = [];
                querySnapshot.forEach((doc) => {
                    players.push({ id: doc.id, ...doc.data() });
                });
                renderUI(players);
                loadingOverlay.style.display = 'none';
            });
        }
        
        function renderUI(players) {
            playerList.innerHTML = '';
            screenshotListEl.innerHTML = '';
            let team1Score = 0, team2Score = 0;
            let hasScreenshots = false;

            noPlayersMessage.style.display = players.length === 0 ? 'block' : 'none';
            
            const sortedPlayers = [...players].sort((a, b) => b.score - a.score);

            sortedPlayers.forEach(player => {
                if (player.team == 1) team1Score += player.score;
                if (player.team == 2) team2Score += player.score;

                const canEdit = isAdmin || (player.claimedBy === currentUser.uid);

                const playerCard = document.createElement('div');
                const teamBorderClass = player.team == 1 ? 'border-blue-700' : 'border-red-700';
                
                playerCard.className = `bg-gray-800 p-5 rounded-xl shadow-lg border ${teamBorderClass} flex flex-col justify-between`;
                
                let adminDeleteButton = isAdmin ? `<button onclick="window.removePlayer('${player.id}')" class="text-gray-500 hover:text-red-500 text-xs font-bold">삭제</button>` : '';

                let scoreControlsHTML = '';
                if(canEdit) {
                     scoreControlsHTML = `
                        <div class="grid grid-cols-2 gap-3 mt-4">
                            <button onclick="window.updateScore('${player.id}', 'WIN', ${player.score}, ${player.games})" class="btn bg-blue-600 hover:bg-blue-500 text-white font-semibold py-2 rounded-md">승</button>
                            <button onclick="window.updateScore('${player.id}', 'LOSS', ${player.score}, ${player.games})" class="btn bg-red-600 hover:bg-red-500 text-white font-semibold py-2 rounded-md">패</button>
                            <button onclick="window.updateScore('${player.id}', 'SNIPE_WIN', ${player.score}, ${player.games})" class="btn col-span-2 bg-purple-600 hover:bg-purple-500 text-white font-semibold py-2 rounded-md">저격 승</button>
                        </div>`;
                } else if(!player.claimedBy) {
                     scoreControlsHTML = `<button onclick="window.claimPlayer('${player.id}')" class="btn w-full bg-green-600 hover:bg-green-500 text-white font-semibold py-2 rounded-md mt-4">이건 내 아이디!</button>`;
                } else {
                    scoreControlsHTML = `<div class="text-center text-sm text-gray-500 py-2 mt-4">(${player.claimedBy.substring(0, 6)}... 님이 관리중)</div>`;
                }

                let historyHTML = '<p class="text-xs text-gray-500">기록 없음</p>';
                if (player.history && player.history.length > 0) {
                    const sortedHistory = [...player.history].sort((a, b) => (b.timestamp?.seconds || 0) - (a.timestamp?.seconds || 0));
                    historyHTML = sortedHistory.map(log => {
                        const logType = log.type === 'WIN' ? '승' : (log.type === 'LOSS' ? '패' : '저격승');
                        const color = log.scoreChange > 0 ? 'text-green-400' : 'text-red-400';
                        return `<p class="border-b border-gray-700 py-1 ${color}">${logType} (${log.scoreChange > 0 ? '+' : ''}${log.scoreChange}점)</p>`;
                    }).join('');
                }

                let screenshotHTML = '';
                 if (canEdit) {
                    screenshotHTML = `
                    <div class="mt-4">
                        <label class="text-xs font-semibold text-gray-300 mb-1 block">스크린샷 URL</label>
                        <div class="flex gap-2">
                            <input type="url" id="screenshot-url-${player.id}" value="${player.screenshotUrl || ''}" placeholder="https://..." class="w-full bg-gray-700 text-white text-xs rounded-md px-2 py-1 border border-gray-600 focus:outline-none focus:ring-1 focus:ring-cyan-500">
                            <button onclick="window.updateScreenshot('${player.id}')" class="btn bg-gray-600 hover:bg-gray-500 text-white text-xs font-bold py-1 px-2 rounded-md">저장</button>
                        </div>
                    </div>`;
                }
                
                playerCard.innerHTML = `
                    <div>
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="text-xl font-bold text-cyan-400">${player.name}</h3>
                                <p class="text-sm text-gray-400">${player.tier}</p>
                            </div>
                            ${adminDeleteButton}
                        </div>
                        <div class="text-center my-4">
                            <p class="text-sm text-gray-300">현재 점수</p>
                            <p class="score-display text-white">${player.score}</p>
                            <p class="text-xs text-gray-400 mt-1">총 ${player.games}판</p>
                        </div>
                        <div class="mt-4">
                            <h4 class="text-sm font-semibold text-gray-300 mb-2">게임 내역</h4>
                            <div class="game-log max-h-24 overflow-y-auto bg-gray-900 rounded-md p-2 text-xs">${historyHTML}</div>
                        </div>
                    </div>
                    <div>
                        ${scoreControlsHTML}
                        ${screenshotHTML}
                    </div>
                `;
                playerList.appendChild(playerCard);

                if(player.screenshotUrl) {
                    hasScreenshots = true;
                    const screenshotItem = document.createElement('div');
                    screenshotItem.className = 'flex justify-between items-center py-2 border-b border-gray-700';
                    screenshotItem.innerHTML = `
                        <p class="text-white">${player.name}</p>
                        <a href="${player.screenshotUrl}" target="_blank" rel="noopener noreferrer" class="text-cyan-400 hover:text-cyan-300 font-semibold text-sm">링크 보기</a>
                    `;
                    screenshotListEl.appendChild(screenshotItem);
                }
            });
            
            noScreenshotsMessage.style.display = hasScreenshots ? 'none' : 'block';
            if (hasScreenshots) screenshotListEl.prepend(noScreenshotsMessage);

            team1ScoreEl.textContent = team1Score;
            team2ScoreEl.textContent = team2Score;
        }
        
        window.updateScore = async (playerId, type, currentScore, currentGames) => {
            const playerRef = doc(db, `artifacts/${appId}/public/data/players`, playerId);
            const scoreChange = SCORE_RULES[type];
            await updateDoc(playerRef, {
                score: currentScore + scoreChange,
                games: currentGames + 1,
                history: arrayUnion({ type, scoreChange, timestamp: new Date() })
            });
        };
        
        window.removePlayer = async (playerId) => {
            if (confirm(`정말 이 참가자를 삭제하시겠습니까?`)) {
                 const playerRef = doc(db, `artifacts/${appId}/public/data/players`, playerId);
                 await deleteDoc(playerRef);
            }
        };

        window.claimPlayer = async (playerId) => {
            if(confirm('이 참가자를 본인으로 선택하시겠습니까? 선택 후에는 다른 사람이 선택할 수 없습니다.')){
                 const playerRef = doc(db, `artifacts/${appId}/public/data/players`, playerId);
                 await updateDoc(playerRef, { claimedBy: currentUser.uid });
            }
        };
        
        window.updateScreenshot = async (playerId) => {
            const inputEl = document.getElementById(`screenshot-url-${playerId}`);
            const url = inputEl.value.trim();
            const playerRef = doc(db, `artifacts/${appId}/public/data/players`, playerId);
            await updateDoc(playerRef, { screenshotUrl: url });
            alert('스크린샷 URL이 저장되었습니다.');
        };

        addPlayerForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!isAdmin) return;
            const newPlayer = {
                name: playerNameInput.value.trim(),
                tier: playerTierInput.value.trim(),
                team: parseInt(playerTeamSelect.value),
                score: 0,
                games: 0,
                createdAt: serverTimestamp(),
                claimedBy: null,
                history: [],
                screenshotUrl: ''
            };
            await addDoc(collection(db, `artifacts/${appId}/public/data/players`), newPlayer);
            addPlayerForm.reset();
        });
    </script>
</body>
</html>

