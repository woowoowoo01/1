<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, maximum-scale=5.0">
    <title>선거 공보물 및 신문 기사</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Ensure html and body take full viewport width and prevent horizontal scrolling */
        html, body {
            margin: 0;
            padding: 0;
            width: 100vw; /* Explicitly set to full viewport width */
            overflow-x: hidden; /* Prevent horizontal scrolling */
        }

        /* 기본 폰트 설정 및 배경색 */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #F8F8F8; /* Light Gray */
            color: #333333; /* Dark Gray */
            line-height: 1.6;
            min-height: 100vh; /* 최소 뷰포트 높이만큼 확장 */
            display: flex;
            flex-direction: column;
        }

        /* 스크롤바 스타일링 */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #E0E0E0;
        }
        ::-webkit-scrollbar-thumb {
            background: #B0B0B0;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #909090;
        }

        /* 네비게이션 버튼 기본 스타일 */
        .nav-item {
            transition: background-color 0.3s ease, color 0.3s ease;
            white-space: nowrap; /* Prevent text wrapping */
            flex-shrink: 0; /* Prevent buttons from shrinking too much */
        }
        .nav-item.active {
            font-weight: bold;
        }

        /* 공약 카드 및 기사 카드 공통 스타일 */
        .card {
            background-color: #FFFFFF;
            border-left: 5px solid; /* Dynamic color */
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }
        .card-header {
            padding: 1rem;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .card-content {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.5s ease-out, padding 0.5s ease-out;
            padding-left: 1rem;
            padding-right: 1rem;
            padding-bottom: 0; /* Initial padding-bottom */
        }
        .card-content.open {
            max-height: 1000px; /* Sufficiently large value */
            padding-bottom: 1rem;
        }
        .card-header span {
            transition: transform 0.3s ease;
        }
        .card-content.open + .card-header span {
            transform: rotate(180deg);
        }

        /* 후보자별 색상 변수 (JavaScript에서 동적으로 설정) */
        :root {
            --candidate-primary-color: #4CAF50; /* Default Green */
            --candidate-secondary-color: #429946; /* Default Darker Green */
            --candidate-background-color: #E8F5E9; /* Default Light Green */
            --candidate-text-color: #2E7D32; /* Default Dark Green */
            --candidate-card-bg-color: #FFFFFF; /* Default White */
        }

        .header-bg {
            background-color: var(--candidate-primary-color);
        }
        .header-text-color {
            color: var(--candidate-background-color);
        }
        .bio-bg {
            background-color: var(--candidate-card-bg-color);
            color: var(--candidate-text-color);
        }
        .nav-bg {
            background-color: var(--candidate-secondary-color);
        }
        .nav-item.active {
            background-color: var(--candidate-primary-color);
            color: #FFFFFF;
        }
        .nav-item:hover {
            background-color: var(--candidate-primary-color);
        }
        .pledge-card {
            border-left-color: var(--candidate-primary-color);
            background-color: var(--candidate-card-bg-color);
        }
        .footer-bg {
            background-color: var(--candidate-text-color);
            color: var(--candidate-background-color);
        }

        /* 메인 포스터 화면 스타일 */
        #main-poster-screen {
            background: linear-gradient(135deg, #a8dadc, #457b9d); /* Gradient background */
            color: white;
            text-align: center;
            flex-grow: 1; /* 남은 공간을 채우도록 설정 */
            display: flex; /* Ensure it's a flex container */
            flex-direction: column;
            justify-content: center;
            align-items: center;
            width: 100%; /* Ensure it takes full width */
            padding: 2rem 0; /* Add back vertical padding, no horizontal */
        }

        /* Inner content div for main poster screen to apply max-width and centering */
        #main-poster-screen > div {
            max-width: 1280px; /* Equivalent to Tailwind's max-w-7xl */
            width: 100%;
            margin-left: auto;
            margin-right: auto;
            padding-left: 1rem; /* px-4 */
            padding-right: 1rem; /* px-4 */
        }
        @media (min-width: 640px) { /* sm:px-8 */
            #main-poster-screen > div {
                padding-left: 2rem;
                padding-right: 2rem;
            }
        }
        @media (min-width: 1024px) { /* lg:px-16 */
            #main-poster-screen > div {
                padding-left: 4rem;
                padding-right: 4rem;
            }
        }

        /* Content area styles */
        #content-area {
            padding-top: 4rem; /* Offset for fixed nav bar (h-16 = 64px = 4rem) */
            overflow-y: auto;
            flex-grow: 1; /* 남은 공간을 채우도록 설정 */
            width: 100%; /* Ensure it takes full width, content inside will be max-width */
            display: none; /* Initially hidden */
            flex-direction: column; /* Ensure it's a flex column when visible */
        }
        /* Inner content div for content area to apply max-width and centering */
        #content-area > div {
            max-width: 1280px; /* Equivalent to max-w-7xl */
            margin-left: auto;
            margin-right: auto;
            padding-left: 1rem; /* px-4 */
            padding-right: 1rem; /* px-4 */
        }
        @media (min-width: 640px) { /* sm:px-8 */
            #content-area > div {
                padding-left: 2rem;
                padding-right: 2rem;
            }
        }
        @media (min-width: 1024px) { /* lg:px-16 */
            #content-area > div {
                padding-left: 4rem;
                padding-right: 4rem;
            }
        }


        .candidate-grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 1.5rem;
            margin-top: 2rem;
        }
        @media (min-width: 640px) {
            .candidate-grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }
        @media (min-width: 1024px) {
            .candidate-grid {
                grid-template-columns: repeat(4, 1fr);
            }
        }
        .candidate-poster-card {
            background-color: rgba(255, 255, 255, 0.15);
            border-radius: 0.75rem;
            padding: 1.5rem;
            backdrop-filter: blur(5px);
            border: 1px solid rgba(255, 255, 255, 0.3);
            transition: transform 0.2s ease-in-out, background-color 0.2s ease-in-out;
            cursor: pointer;
        }
        .candidate-poster-card:hover {
            transform: translateY(-5px);
            background-color: rgba(255, 255, 255, 0.25);
        }

        /* Transition overlay styles */
        .transition-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2.5rem;
            font-weight: bold;
            z-index: 1000;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.5s ease-in-out, visibility 0.5s ease-in-out;
            text-align: center; /* Center text for line break */
        }
        .transition-overlay.active {
            opacity: 1;
            visibility: visible;
        }


        /* Balance Game specific styles */
        .balance-game-container {
            background: linear-gradient(135deg, #e0f7fa, #b2ebf2); /* Light, vibrant blue gradient */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: calc(100vh - 128px); /* Adjust for header/footer */
            text-align: center;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
        }
        .question-card {
            background-color: #FFFFFF;
            border-radius: 0.75rem;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
            padding: 2.5rem;
            margin-bottom: 2rem;
            max-width: 700px;
            width: 100%;
            border: 2px solid #81d4fa; /* Light blue border */
        }
        .option-button {
            background-color: #4CAF50; /* Green */
            color: white;
            padding: 1rem 2rem;
            border-radius: 0.5rem;
            font-size: 1.25rem;
            font-weight: bold;
            transition: background-color 0.3s ease, transform 0.1s ease;
            width: 100%;
            margin-bottom: 1rem;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            border: none; /* Remove default button border */
        }
        .option-button:hover {
            background-color: #45a049;
            transform: translateY(-2px);
        }
        .option-button:active {
            transform: translateY(0);
        }
        .option-button.selected {
            background-color: #2196F3; /* Blue */
            box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.5);
        }
        .result-card {
            background-color: #FFFFFF;
            border-radius: 0.75rem;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
            padding: 2.5rem;
            max-width: 700px;
            width: 100%;
            text-align: center;
            border: 2px solid #81d4fa; /* Light blue border */
        }
        .start-button {
            background-color: #007BFF; /* Blue */
            color: white;
            padding: 1.25rem 3rem;
            border-radius: 0.75rem;
            font-size: 1.5rem;
            font-weight: bold;
            transition: background-color 0.3s ease, transform 0.1s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border: none; /* Remove default button border */
        }
        .start-button:hover {
            background-color: #0056b3;
            transform: translateY(-2px);
        }

        /* New styles for balance game result page */
        #result-area {
            background: linear-gradient(160deg, #e6ffed, #d0f0d0); /* Soft green gradient */
            padding: 2.5rem;
            border-radius: 1rem;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
            color: #2e7d32; /* Dark green text */
        }
        #result-area h2 {
            color: #1b5e20; /* Even darker green for heading */
            font-size: 2.5rem;
            margin-bottom: 1.5rem;
        }
        #result-area #result-text {
            font-size: 1.2rem;
            line-height: 1.8;
            text-align: left; /* Align text left for readability */
            margin-bottom: 2rem;
            background-color: rgba(255, 255, 255, 0.7);
            padding: 1.5rem;
            border-radius: 0.75rem;
            border: 1px solid #a5d6a7;
            white-space: pre-wrap; /* Preserve line breaks from JS */
        }
        #result-area #restart-balance-game {
            background-color: #4CAF50; /* Green button */
        }
        #result-area #restart-balance-game:hover {
            background-color: #388E3C; /* Darker green on hover */
        }
    </style>
</head>
<body class="antialiased min-h-screen flex flex-col">

    <nav class="bg-gray-800 text-white shadow-lg sticky top-0 z-50 w-full">
        <div class="mx-auto px-2 sm:px-4 lg:px-6 flex items-center h-16 w-full">
            <div class="flex-shrink-0 mr-4">
                <span class="text-2xl font-bold">선거 공보물</span>
            </div>
            <div class="flex-grow overflow-x-auto">
                <div class="flex items-center space-x-4 py-2">
                    <button id="nav-main" data-view="main" class="nav-item px-3 py-2 rounded-md text-sm font-medium active">메인 화면</button>
                    <button id="nav-kanghanul" data-view="kanghanul" class="nav-item px-3 py-2 rounded-md text-sm font-medium">기호 1번 강한울</button>
                    <button id="nav-choijia" data-view="choijia" class="nav-item px-3 py-2 rounded-md text-sm font-medium">기호 2번 최지아</button>
                    <button id="nav-parksunwoo" data-view="parksunwoo" class="nav-item px-3 py-2 rounded-md text-sm font-medium">기호 3번 박선우</button>
                    <button id="nav-yoonnarae" data-view="yoonnarae" class="nav-item px-3 py-2 rounded-md text-sm font-medium">기호 4번 윤나래</button>
                    <button id="nav-news" data-view="news" class="nav-item px-3 py-2 rounded-md text-sm font-medium">📰 신문 기사</button>
                    <button id="nav-balance-game" data-view="balance-game" class="nav-item px-3 py-2 rounded-md text-sm font-medium">⚖️ 밸런스 게임</button>
                </div>
            </div>
        </div>
    </nav>

    <div id="main-poster-screen" class="flex-grow flex flex-col justify-center items-center w-full">
        <div class="max-w-7xl mx-auto px-4 sm:px-8 lg:px-16 w-full text-white text-center">
            <h1 class="text-5xl font-extrabold mb-4 drop-shadow-lg">2025<br>미래를 위한 선택</h1>
            <p class="text-2xl font-semibold mb-8 drop-shadow-md">당신의 한 표가 대한민국의 내일을 만듭니다</p>

            <div class="candidate-grid w-full">
                <div class="candidate-poster-card" data-candidate-key="kanghanul">
                    <p class="text-4xl font-bold mb-2" style="color: #E2725B;">1</p>
                    <h3 class="text-2xl font-bold">강한울</h3>
                    <p class="text-lg">내일당</p>
                </div>
                <div class="candidate-poster-card" data-candidate-key="choijia">
                    <p class="text-4xl font-bold mb-2" style="color: #6A5ACD;">2</p>
                    <h3 class="text-2xl font-bold">최지아</h3>
                    <p class="text-lg">푸른하늘당</p>
                </div>
                <div class="candidate-poster-card" data-candidate-key="parksunwoo">
                    <p class="text-4xl font-bold mb-2" style="color: #4169E1;">3</p>
                    <h3 class="text-2xl font-bold">박선우</h3>
                    <p class="text-lg">미래로당</p>
                </div>
                <div class="candidate-poster-card" data-candidate-key="yoonnarae">
                    <p class="text-4xl font-bold mb-2" style="color: #8FBC8F;">4</p>
                    <h3 class="text-2xl font-bold">윤나래</h3>
                    <p class="text-lg">함께해요당</p>
                </div>
            </div>
        </div>
    </div>

    <div id="content-area" class="flex-grow py-10 w-full">
        <div class="max-w-7xl mx-auto px-4 sm:px-8 lg:px-16"></div>
    </div>

    <div id="transition-overlay" class="transition-overlay">
        <span id="transition-message"></span>
    </div>

    <script>
        // 후보자 데이터
        const candidatesData = {
            "kanghanul": {
                name: "강한울",
                party: "내일당",
                slogan: "모두의 내일을 위한, 과감한 변화!",
                bio: "안녕하세요, 내일당 후보 강한울입니다. 저는 우리 사회에서 어려움을 겪는 친구들을 돕고, 모두가 함께 행복하게 잘 사는 세상을 만들고 싶습니다. 저는 새로운 변화를 두려워하지 않고, 여러분의 더 밝은 내일을 위해 힘껏 노력할 준비가 되어있습니다!",
                career: "명문대 졸업 후 대기업 금융 부서에서 근무하며 경제 전문가로 성장했습니다. 이후 시민단체 활동을 통해 사회 문제에 눈을 뜨고, 국회 보좌관을 거쳐 정치에 입문했습니다. 현재는 내일당의 정책위 부의장으로 활동하며 실물 경제와 정책 실무에 대한 깊은 이해를 바탕으로 활발한 의정 활동을 펼치고 있습니다.",
                colors: {
                    primary: "#E2725B", /* Header, active nav, border-left */
                    secondary: "#d1614a", /* Nav background, hover */
                    background: "#F5F5DC", /* Body background */
                    text: "#5D4037", /* Body text */
                    cardBg: "#FFF8DC" /* Card background */
                },
                pledges: [
                    { category: "economy", icon: "📈", title: "기업 투자 활성화 및 세금 감면으로 경제 성장 가속화", details: ["기업이 사업하기 좋게 세금을 깎아주고 불필요한 규칙을 없애서 투자를 많이 하게 하겠습니다. 그러면 경제가 활발해지고 일자리도 늘어날 것입니다.", "큰 회사와 작은 회사가 서로 돕도록 지원하고, 세금을 줄여서 사람들이 돈을 더 많이 쓸 수 있게 하겠습니다. 이 돈으로 청년 일자리를 만들고 경제를 성장시킬 것입니다."] },
                    { category: "welfare", icon: "🎯", title: "선택적 복지 강화 및 자립 지원을 통한 복지 효율성 증대", details: ["정말 도움이 필요한 사람들에게만 집중해서 복지 혜택을 줄 것입니다. 복지 예산이 낭비되지 않게 해서 아낀 돈으로 어려운 사람들을 더 많이 돕겠습니다.", "스스로 돈 벌고 살 수 있도록 교육과 직업 훈련을 늘리겠습니다. 건강보험도 낭비 없이 잘 운영해서 아플 때 걱정 없이 병원에 갈 수 있게 할 것입니다."] },
                    { category: "security", icon: "💪", title: "첨단 국방력 강화 및 한미동맹 굳건화를 통한 강력한 안보 태세 확립", details: ["최신 무기를 개발하고 미국과의 동맹을 튼튼히 해서 나라를 든든하게 지키겠습니다. 국방 예산을 아끼고 무기 수출을 늘려서 돈을 마련할 것입니다.", "북한 핵 문제를 해결하고, 군인들이 더 좋은 환경에서 근무하고 전역 후에도 잘 지낼 수 있도록 지원을 늘리겠습니다."] },
                    { category: "culture", icon: "📚", title: "자율형 사립고 확대 및 경쟁 시스템 도입으로 교육 수월성 제고", details: ["학교가 자유롭게 교육하고 학생들이 경쟁하며 실력을 키울 수 있게 하겠습니다. 학교마다 특색 있는 교육을 하고 사립 학교의 자율성을 높여 교육의 다양성을 늘릴 것입니다.", "대학 입시를 더 공정하게 만들고, 인공지능 같은 새 기술을 가르쳐서 미래에 필요한 인재를 키울 것입니다."] },
                    { category: "environment", icon: "🌳", title: "시장 메커니즘 활용한 친환경 기술 개발 투자 유도 및 규제 합리화", details: ["기업들이 친환경 기술을 개발하도록 세금 혜택을 주고, 정부가 직접 규제하기보다는 기업이 자율적으로 환경을 개선하도록 돕겠습니다. 탄소 배출을 줄이는 데 시장 원리를 활용할 것입니다.", "미세먼지를 줄이는 기술 개발을 지원하고, 국민들이 스스로 환경 보호에 참여하도록 독려할 것입니다."] },
                    { category: "economy", icon: " ", title: "재건축·재개발 규제 완화 및 민간 공급 확대로 주택 시장 안정화", details: ["낡은 아파트를 새로 짓는 규제를 풀고, 민간 건설사가 집을 많이 짓게 해서 집값을 안정시키겠습니다.", "전세 사기를 없애기 위한 법을 강화하고, 부동산 거래 정보를 투명하게 공개해서 세입자를 보호할 것입니다."] },
                    { category: "economy", icon: "🧑‍🏭", title: "유연한 노동 시장 구축 및 성과 중심 보상 체계 확립으로 생산성 향상", details: ["딱딱한 노동 규칙을 바꾸고, 일한 만큼 돈을 더 많이 받을 수 있게 해서 생산성을 높이겠습니다.", "회사와 직원들이 자유롭게 협상하고, 새로운 형태의 일자리(플랫폼 노동 등)에 맞는 유연한 정책을 만들 것입니다."] },
                    { category: "public", icon: "🏛️", title: "공공 부문 효율화 및 작은 정부 지향으로 재정 건전성 확보", details: ["정부의 불필요한 지출을 줄이고 공무원 수를 줄여서 '작은 정부'를 만들 것입니다. 국민 세금이 낭비되지 않게 하겠습니다.", "공공 기관의 낭비를 막고 예산 사용을 투명하게 공개해서 국민들이 정부를 더 믿을 수 있게 할 것입니다."] },
                    { category: "economy", icon: "🌐", title: "자유무역 확대 및 국제 경쟁력 강화를 통한 수출 증대", details: ["다른 나라와 자유롭게 무역할 수 있는 협정을 더 많이 맺고, 우리 기업이 해외로 진출하는 것을 적극 돕겠습니다. 수출을 늘려서 경제를 성장시킬 것입니다.", "핵심 기술을 보호하고 해외 투자를 유치해서 좋은 일자리를 만들 것입니다."] },
                    { category: "justice", icon: "⚖️", title: "법치주의 확립 및 강력 범죄 처벌 강화를 통한 사회 질서 유지", details: ["법을 엄격하게 적용하고 강력 범죄를 강하게 처벌해서 국민들이 안전하게 살 수 있게 하겠습니다.", "경찰과 검찰의 수사 능력을 키우고, 디지털 범죄나 마약 범죄 같은 새로운 범죄에 잘 대응할 것입니다."] }
                ]
            },
            "choijia": {
                name: "최지아",
                party: "푸른하늘당",
                slogan: "불평등 없는 세상, 모두가 존엄한 정의로운 미래!", /* Justice Party ideology */
                bio: "안녕하세요, 푸른하늘당 후보 최지아입니다. 저는 우리 사회에서 어려움을 겪는 친구들을 돕고, 모두가 함께 행복하게 잘 사는 세상을 만들고 싶습니다. 저는 새로운 변화를 두려워하지 않고, 여러분의 더 밝은 내일을 위해 힘껏 노력할 준비가 되어있습니다!",
                career: "사회학과를 졸업하고 노동 운동 현장에서 약자의 목소리에 귀 기울여 왔습니다. 인권 변호사로 활동하며 소외된 이들의 권익 보호에 앞장섰고, 시민 사회 운동을 통해 사회 변화를 이끌었습니다. 현재는 푸른하늘당의 대변인으로서 진보적 가치를 확산하고 있습니다.",
                colors: {
                    primary: "#6A5ACD", /* SlateBlue */
                    secondary: "#5A4CBE", /* Darker SlateBlue */
                    background: "#F0F8FF", /* AliceBlue */
                    text: "#2F4F4F", /* DarkSlateGray */
                    cardBg: "#FFFFFF" /* White */
                },
                pledges: [
                    { category: "economy", icon: "✊", title: "비정규직 철폐 및 최저임금 인상으로 노동 존중 사회 실현", details: ["비정규직을 줄이고 최저 임금을 올려서 일하는 사람들이 더 잘 살 수 있게 하겠습니다. 노동조합의 권리도 지켜줄 것입니다.", "큰 회사들이 사회적 책임을 다하고, 작은 가게들을 보호해서 공정한 경쟁을 할 수 있게 할 것입니다."] },
                    { category: "welfare", icon: "💖", title: "모든 시민을 위한 기본소득 도입 검토 및 공공 의료 시스템 대폭 강화", details: ["모든 국민이 기본적인 생활을 할 수 있도록 기본소득 도입을 검토하고, 공공 병원과 의료 서비스를 늘려서 의료비 걱정을 줄일 것입니다.", "정신 건강 상담을 쉽게 받을 수 있게 하고, 정신 질환을 일찍 발견하고 치료할 수 있는 시스템을 강화할 것입니다."] },
                    { category: "security", icon: "🕊️", title: "평화적 비핵화 및 군사적 긴장 완화를 위한 대화와 협력 확대", details: ["북한과 대화하고 협력해서 한반도에서 핵무기를 없애고 평화를 만들겠습니다. 불필요한 군사비는 줄일 것입니다.", "군인들의 인권을 보호하고 복지를 늘릴 것입니다. 군대 내 인권 침해를 막고 상담 시스템을 강화하겠습니다."] },
                    { category: "culture", icon: "🎓", title: "무상 교육 전면 확대 및 입시 경쟁 완화를 통한 공평한 교육 기회 보장", details: ["경쟁보다는 학생들의 창의성을 키우는 교육으로 바꾸고, 교육비 걱정 없이 학교 다닐 수 있게 무상 교육을 늘리겠습니다.", "다양한 배경을 가진 학생들이 차별 없이 교육받고, 특수 교육이 필요한 학생들을 위한 맞춤 교육을 강화할 것입니다."] },
                    { category: "environment", icon: "♻️", title: "탄소세 도입 및 재생에너지 전환 가속화를 통한 기후 위기 최우선 대응", details: ["기후 변화에 적극적으로 대응하기 위해 탄소세를 도입하고, 태양광이나 풍력 같은 재생 에너지를 더 많이 사용할 것입니다.", "기업들이 환경 오염에 책임지게 하고, 시민들이 환경 보호에 참여하도록 교육과 지원을 늘릴 것입니다."] },
                    { category: "economy", icon: "🏡", title: "공공 주택 대폭 확대 및 부동산 투기 이익 환수로 주거권 보장", details: ["집은 모든 사람이 살 권리가 있는 곳입니다. 싸고 좋은 공공 주택을 많이 지어서 집 걱정 없이 살 수 있게 하겠습니다.", "부동산 투기를 철저히 막고, 투기로 번 돈은 다시 사회로 돌려받을 것입니다."] },
                    { category: "economy", icon: "🗓️", title: "주 4일 근무제 도입 검토 및 노동조합 활동 보장으로 노동시간 단축 및 삶의 질 향상", details: ["일하는 사람들의 권리를 지키고, 주 4일 근무제를 검토해서 더 여유로운 삶을 살 수 있게 하겠습니다.", "노동조합이 자유롭게 활동하도록 돕고, 회사와 직원들이 서로 협력해서 좋은 노동 문화를 만들 것입니다."] },
                    { category: "gender", icon: "🚻", title: "성평등 사회 구현 및 성차별 해소를 위한 노력 강화", details: ["성별에 따른 임금 차이를 없애고, 성폭력과 디지털 성범죄를 강력히 처벌하며 피해자를 돕겠습니다.", "성평등 교육을 의무화하고, 성차별적인 문화와 관행을 개선하는 캠페인을 늘릴 것입니다."] },
                    { category: "social", icon: "🌱", title: "사회적 기업 육성 및 사회적 경제 활성화를 통한 포용적 성장", details: ["사회적 가치를 만드는 기업(사회적 기업)을 키우고, 협동조합이나 마을 기업을 많이 만들어서 어려운 사람들에게 일자리를 줄 것입니다.", "공정 무역을 장려하고, 기업들이 사회적 책임을 다하도록 해서 지속 가능한 사회를 만들 것입니다."] },
                    { category: "animal", icon: "🐾", title: "동물권 강화 및 동물 복지 증진을 위한 법·제도 개선", details: ["동물 학대 처벌을 강화하고, 버려진 동물들을 더 잘 보호하겠습니다. 반려동물 등록을 의무화하고 입양을 장려할 것입니다.", "동물 실험을 줄이고 동물 복지를 생각하는 축산 방식을 장려할 것입니다. 동물원과 수족관의 동물 복지 기준도 높일 것입니다."] }
                ]
            },
            "parksunwoo": {
                name: "박선우",
                party: "미래로당",
                slogan: "새로운 생각으로, 내일을 디자인하다!",
                bio: "안녕하세요, 미래로당 후보 박선우입니다. 저는 젊은 세대의 목소리에 귀 기울이고, 합리적인 생각으로 미래를 준비하는 사람입니다. 변화를 두려워하지 않고, 새로운 아이디어로 우리 사회를 더 멋지게 디자인하고자 합니다.",
                career: "과학고와 카이스트를 졸업한 이공계 출신으로, 스타트업을 창업하여 성공적으로 이끌었던 젊은 기업가입니다. 4차 산업혁명 기술에 대한 깊은 이해를 바탕으로 정부의 혁신 자문 위원으로 활동했으며, 젊은 세대의 목소리를 대변하고자 정치에 뛰어들었습니다. 현재는 미래로당의 청년 위원장으로 활동하며 미래 지향적인 정책을 제시하고 있습니다.",
                colors: {
                    primary: "#4169E1", /* RoyalBlue */
                    secondary: "#365BCB", /* Darker RoyalBlue */
                    background: "#E0FFFF", /* LightCyan */
                    text: "#4682B4", /* SteelBlue */
                    cardBg: "#FFFFFF" /* White */
                },
                pledges: [
                    { category: "economy", icon: "🚀", title: "인공지능·바이오 등 미래 신산업 육성 및 스타트업 전폭 지원", details: ["인공지능, 바이오, 우주 산업 같은 미래 산업을 키우고, 새로운 아이디어로 시작하는 회사(스타트업)를 많이 지원하겠습니다. 투자와 세금 혜택을 늘릴 것입니다.", "인공지능, 빅데이터 같은 새 기술 분야의 전문가를 키우고, 관련 산업을 지원해서 새로운 일자리를 많이 만들 것입니다."] },
                    { category: "welfare", icon: "🎯", title: "빅데이터 기반 맞춤형 복지 시스템 구축으로 복지 사각지대 해소", details: ["빅데이터를 활용해서 정말 도움이 필요한 사람들을 찾아서 딱 맞는 복지 서비스를 줄 것입니다. 복지 예산을 효율적으로 쓸 것입니다.", "젊은 세대가 창업하고 일자리를 쉽게 찾을 수 있도록 돕고, 집 걱정을 덜어주고 재산을 모을 수 있게 지원할 것입니다."] },
                    { category: "security", icon: "🤖", title: "첨단 과학기술 기반 스마트 국방 시스템 구축 및 국방 효율성 증대", details: ["최신 과학 기술로 군대를 스마트하게 만들고, 불필요한 국방 예산을 줄이겠습니다. 인공지능 드론 같은 첨단 무기로 군사력을 강화할 것입니다.", "군인들의 월급을 올리고 자기 계발 기회를 늘려서, 젊은 세대가 가고 싶은 군대를 만들겠습니다. 전역 후에도 잘 지낼 수 있게 도울 것입니다."] },
                    { category: "culture", icon: "💻", title: "초등 코딩 교육 의무화 및 AI 교육 강화로 미래 시대 인재 양성", details: ["어릴 때부터 코딩 교육을 의무화하고 인공지능 교육을 강화해서 미래에 필요한 인재를 키우겠습니다. 단순히 지식만 가르치는 게 아니라 문제 해결 능력을 키울 것입니다.", "학생들이 스스로 꿈을 찾고 진로를 결정할 수 있도록 다양한 직업 체험과 맞춤형 진로 교육을 제공할 것입니다."] },
                    { category: "environment", icon: "🌱", title: "과학적 데이터 기반 환경 관리 및 미세먼지 저감 기술 개발 투자", details: ["과학적인 데이터와 인공지능 기술로 환경 오염을 예측하고 해결하겠습니다. 미세먼지를 줄이는 기술 개발에 투자할 것입니다.", "수소 에너지, 소형 원자로 같은 깨끗한 차세대 에너지 기술 개발을 지원하고, 관련 기업에 세금 혜택을 줄 것입니다."] },
                    { category: "economy", icon: "🏘️", title: "청년 공유 주택 확대 및 스마트 기술 활용 주거 환경 개선", details: ["젊은 사람들이 월세 걱정 없이 살 수 있도록 공유 주택을 늘리고, 싸고 좋은 공공 임대 주택을 많이 지을 것입니다.", "부동산 거래를 투명하게 만들고, 불법 투기를 강력히 처벌해서 건강한 주택 시장을 만들 것입니다."] },
                    { category: "economy", icon: "📊", title: "직무 중심 보상 체계 확립 및 유연 근무 확산으로 생산성 및 워라밸 동시 추구", details: ["일한 만큼 보상받는 시스템을 만들고, 유연 근무를 늘려서 생산성과 워라밸(일과 삶의 균형)을 동시에 잡겠습니다.", "회사와 직원들이 자유롭게 협상하고, 플랫폼 노동 같은 새로운 일자리에 맞는 유연한 정책을 만들 것입니다."] },
                    { category: "innovation", icon: "💡", title: "규제 샌드박스 확대 및 신기술 실증 특례 도입으로 혁신 성장 가속화", details: ["새로운 기술이나 서비스가 시장에 쉽게 나올 수 있도록 '규제 샌드박스'를 확대하고, 빨리 상용화될 수 있게 돕겠습니다.", "인공지능, 블록체인 같은 핵심 기술 개발에 투자하고 전문가를 키워서 기술 경쟁력을 높일 것입니다."] },
                    { category: "governance", icon: "📱", title: "디지털 정부 혁신 및 블록체인 기반 행정 서비스 도입으로 효율성·투명성 제고", details: ["정부 업무를 디지털화하고 블록체인 기술을 도입해서 더 투명하고 효율적으로 일하겠습니다. 모바일로 쉽게 민원 처리도 할 수 있게 할 것입니다.", "공공 데이터를 공개하고 빅데이터로 분석해서 국민에게 필요한 정책을 만들 것입니다."] },
                    { category: "space", icon: "🌌", title: "우주 산업 육성 및 민간 우주 개발 지원으로 미래 성장 동력 확보", details: ["우주 발사체와 위성 산업을 키우고, 민간 기업의 우주 개발을 적극 지원해서 미래 성장 동력을 만들겠습니다.", "우주 분야 전문가를 키우고 우주 관련 스타트업을 지원해서 우리나라가 우주 산업을 이끌게 할 것입니다."] }
                ]
            },
            "yoonnarae": {
                name: "윤나래",
                party: "함께해요당",
                slogan: "국민과 함께, 더 나은 내일을 만드는 민생 정치!", /* Democratic Party ideology */
                bio: "안녕하세요, 함께해요당 후보 윤나래입니다. 저는 우리 사회의 모든 사람이 존중받고, 차별 없이 행복하게 살 수 있는 세상을 꿈꾸는 사람입니다. 특히 약한 친구들의 목소리에 귀 기울이고, 따뜻한 마음으로 모두가 함께 살아가는 공동체를 만들고자 합니다.",
                career: "교육대학을 졸업하고 오랫동안 교사로 재직하며 학생들과 소통해왔습니다. 인권 변호사로 활동하며 소외된 이들의 권익 보호에 앞장섰고, 시민 사회 운동을 통해 사회 변화를 이끌었습니다. 현재는 함께해요당의 부대표로서 따뜻한 리더십과 포용의 정치를 실천하고 있습니다.",
                colors: {
                    primary: "#8FBC8F", /* DarkSeaGreen */
                    secondary: "#7BA57B", /* Darker DarkSeaGreen */
                    background: "#E6F0E6", /* Light Greenish-Gray */
                    text: "#4A5A4A", /* Dark Greenish-Gray */
                    cardBg: "#FFFFFF" /* White */
                },
                pledges: [
                    { category: "economy", icon: "🤝", title: "공정 경제 실현 및 소상공인 지원 강화로 경제 민주주의 구현", details: ["큰 회사와 작은 회사 간의 불공정한 거래를 막고, 동네 상권을 살려서 지역 경제를 활성화하겠습니다. 작은 가게들을 위한 정책 자금 지원을 늘릴 것입니다.", "주 4일 근무제 도입을 검토하고, 일하는 사람들의 삶의 질을 높일 것입니다. 노동 시간 단축에 따른 기업 지원도 마련할 것입니다."] },
                    { category: "welfare", icon: "👨‍👩‍👧‍👦", title: "생애 주기별 맞춤형 포용 복지 확대 및 공공 돌봄 시스템 강화", details: ["모든 국민이 태어나서 죽을 때까지 필요한 복지 혜택을 받을 수 있게 하겠습니다. 아이 돌봄 서비스나 노인 돌봄 서비스를 늘릴 것입니다.", "싸고 좋은 공공 임대 주택을 많이 지어서 집 걱정 없이 살 수 있게 하겠습니다. 신혼부부와 청년층을 위한 주거 지원도 강화할 것입니다."] },
                    { category: "peace", icon: "🕊️", title: "한반도 평화 프로세스 재개 및 군사적 신뢰 구축을 통한 평화 정착", details: ["북한과 대화하고 협력해서 한반도에 평화를 가져오겠습니다. 군사적 긴장을 줄이고 비무장지대를 평화적으로 활용할 것입니다.", "군대를 더 효율적으로 만들고, 군인들의 인권을 보호하고 복지를 늘릴 것입니다."] },
                    { category: "culture", icon: "🏫", title: "민주시민 교육 강화 및 공교육 질 향상을 통한 행복한 학교 구현", details: ["입시 경쟁보다는 학생들이 행복하게 성장할 수 있는 교육을 만들겠습니다. 공교육의 질을 높이고 교육 불평등을 줄일 것입니다.", "민주시민 교육을 강화해서 학생들이 공동체 의식을 배우고, 직접 참여하는 교육 프로그램을 늘릴 것입니다."] },
                    { category: "environment", icon: "🌎", title: "국가 주도 재생에너지 전환 로드맵 수립 및 시민 참여형 환경 정책 확대", details: ["기후 변화에 적극적으로 대응하기 위해 정부가 나서서 재생 에너지 사용을 늘리겠습니다. 탄소세 도입도 검토할 것입니다.", "환경 오염을 줄이기 위한 규제를 강화하고, 시민들이 환경 보호에 참여하도록 지원을 늘릴 것입니다."] },
                    { category: "economy", icon: "🏘️", title: "서민 주거 안정 위한 공공 임대 확대 및 전세 사기 근절", details: ["싸고 좋은 공공 임대 주택을 많이 지어서 서민들이 집 걱정 없이 살 수 있게 하겠습니다.", "전세 사기를 없애기 위한 법을 강화하고, 부동산 투기를 철저히 막을 것입니다."] },
                    { category: "economy", icon: "🧑‍💻", title: "노동자 권리 최우선 보장 및 주 4일 근무제 단계적 도입 검토", details: ["일하는 사람들의 권리를 가장 중요하게 생각하고, 주 4일 근무제를 단계적으로 도입해서 삶의 질을 높이겠습니다.", "노동조합 활동을 보장하고, 회사와 직원들이 서로 돕는 문화를 만들 것입니다."] },
                    { category: "region", icon: "🌆", title: "지역 균형 발전 및 지방 소멸 위기 대응을 위한 국가적 투자 확대", details: ["수도권에만 집중된 발전을 지방으로 분산하고, 지역마다 특색 있는 산업을 키워서 지방 경제를 살리겠습니다.", "농어촌에 기본 소득 도입을 검토하고, 귀농·귀촌을 지원해서 지방 소멸 위기에 대응할 것입니다. 지역 의료와 교육 시설도 늘릴 것입니다."] },
                    { category: "health", icon: "🏥", title: "보편적 의료 서비스 강화 및 공공 병원 확충으로 의료 불평등 해소", details: ["모든 국민이 돈 걱정 없이 병원에 갈 수 있도록 의료 서비스를 강화하고, 공공 병원을 늘리겠습니다. 지역 간 의료 격차도 줄일 것입니다.", "아프기 전에 건강을 관리하고, 정신 건강 상담을 쉽게 받을 수 있게 하겠습니다. 만성 질환 관리 시스템도 강화할 것입니다."] },
                    { category: "culture", icon: "🎨", title: "문화 예술 향유 기회 확대 및 문화 복지 강화를 통한 삶의 질 향상", details: ["지역에 문화 시설을 늘리고 문화 바우처를 지원해서 누구나 문화를 즐길 수 있게 하겠습니다. 예술인들의 창작 활동도 도울 것입니다.", "다양한 문화를 존중하고, 소수 문화 예술 단체를 지원하겠습니다. 학교와 지역 사회에서 문화 예술 교육을 늘릴 것입니다."] }
                ]
            }
        };

        // 신문 기사 데이터
        const newspaperArticles = [
            // 강한울 후보 기사 (보수 성향)
            {
                id: "article-kanghanul-1",
                candidateName: "강한울",
                title: "강한울, '기업 자유 보장'으로 경제 활력 회복 선언",
                subtitle: "규제 철폐와 세금 감면, 과감한 정책으로 성장 이끌 것",
                body: `내일당 강한울 후보는 침체된 경제에 활력을 불어넣기 위해 기업 자유 보장과 규제 철폐를 핵심 공약으로 내세웠습니다. 강 후보는 "기업의 자유로운 활동을 보장하고 과도한 규제를 철폐해야 경제 활력을 되찾을 수 있습니다"라고 밝혔습니다. 대기업과 중소기업의 상생 협력을 통해 청년 일자리 창출을 유도하겠다고 덧붙였습니다. 이는 시장 원리에 입각한 성장 전략으로, 보수 경제 정책의 전형을 보여준다는 평가입니다. 그가 제시한 세금 감면과 규제 완화는 기업의 투자 심리를 자극하고, 궁극적으로는 국가 경제의 파이를 키워 모두에게 이익이 돌아갈 것이라는 기대를 모으고 있습니다.`,
                source: "오늘의 진실" // 보수 언론사: 옹호적
            },
            {
                id: "article-kanghanul-2",
                candidateName: "강한울",
                title: "강한울 후보의 '선택적 복지', 사회적 약자 외면 우려",
                subtitle: "재정 건전성 강조 속 복지 사각지대 해소 방안은?",
                body: `내일당 강한울 후보는 '선택과 집중을 통한 효율적 복지 시스템 구축'을 공약하며, "정말 필요한 분들에게 실질적인 도움을 주겠습니다"라고 밝혔습니다. 이는 복지 재정의 건전성을 확보하고 자립을 유도한다는 점에서 긍정적입니다. 그러나 함께해요당 윤나래 후보는 "강한울 후보의 복지 정책은 복지 사각지대를 발생시키고 사회적 약자를 소외시킬 수 있습니다"라고 날 선 비판을 가했습니다. 윤 후보는 "모든 국민이 차별 없이 존중받는 포용적 복지 국가를 지향해야 하며, 강 후보의 정책은 사회적 연대를 약화시킬 수 있습니다"라고 지적하며 복지 확대의 필요성을 강조했습니다.`,
                source: "시민의 눈" // 진보 언론사: 비판적
            },

            // 최지아 후보 기사 (진보 성향)
            {
                id: "article-choijia-1",
                candidateName: "최지아",
                title: "최지아, '보편적 복지 국가'로 모두의 삶 보장",
                subtitle: "생애 주기별 맞춤형 복지 확대, 의료 불평등 해소 기대",
                body: `푸른하늘당 최지아 후보는 '모든 국민을 위한 보편적 복지 국가 실현'을 목표로 생애 주기별 맞춤형 복지 서비스 확대와 공공 의료 시스템 강화를 공약했습니다. 이는 사회적 약자를 보호하고 소득 불균형을 해소하려는 진보적 가치를 담고 있습니다. 최 후보는 "사회적 약자를 위한 주거권 보장을 최우선 과제로 삼겠습니다"라고 밝혔습니다. 포용적 사회 구현에 대한 강한 의지를 드러냈습니다. 그녀의 정책은 사회적 연대를 강화하고 불평등을 해소하는 데 기여할 것이라는 기대를 모으고 있습니다.`,
                source: "미래 탐험 보도" // 진보 언론사: 옹호적
            },
            {
                id: "article-choijia-2",
                candidateName: "최지아",
                title: "최지아 후보의 '과감한 기후 대응', 산업계 부담 가중 우려",
                subtitle: "탄소 배출 규제 강화, 경제 성장과의 균형점은?",
                body: `푸른하늘당 최지아 후보는 기후 위기 대응을 위해 탄소 배출 규제 강화와 재생 에너지 전환 가속화를 핵심 환경 공약으로 내세웠습니다. 이는 환경 보호에 대한 강력한 의지를 보여주지만, 내일당 강한울 후보는 "최지아 후보의 정책은 환경 규제를 강화하여 기업의 생산 비용 증가와 국제 경쟁력 저하를 초래할 수 있습니다"라고 강하게 비판했습니다. 강 후보는 "시장 중심의 환경 정책 추진과 친환경 기술 투자 유도가 필요합니다"라고 주장하며 규제보다는 자율적 기술 개발의 중요성을 강조했습니다.`,
                source: "오늘의 진실" // 보수 언론사: 비판적
            },

            // 박선우 후보 기사 (개혁 보수 성향)
            {
                id: "article-parksunwoo-1",
                candidateName: "박선우",
                title: "박선우, '스마트 혁신'으로 미래 먹거리 창출",
                subtitle: "첨단 과학기술 기반 산업 육성, 스타트업 생태계 조성 기대",
                body: `미래로당 박선우 후보는 '혁신 성장 동력 확보 및 스타트업 생태계 조성'을 통해 "인공지능, 바이오, 우주 산업 등 미래 먹거리를 발굴하겠습니다"라고 약속했습니다. 이는 젊고 혁신적인 정책으로 평가받습니다. 박 후보는 "새로운 아이디어를 가진 젊은이들이 창업해서 성장할 수 있도록 정부가 적극적으로 지원하고, 불필요한 규제를 없애겠습니다"라고 밝혔습니다. 미래 지향적 비전을 제시했습니다. 그의 실용주의적 접근은 급변하는 시대에 필요한 리더십으로 주목받고 있습니다.`,
                source: "미래 탐험 보도" // 진보 언론사: 중립적/긍정적
            },
            {
                id: "article-parksunwoo-2",
                candidateName: "박선우",
                title: "박선우 후보의 '데이터 기반 복지', 프라이버시 침해 우려",
                subtitle: "맞춤형 복지 서비스 제공 속 개인 정보 보호 방안은?",
                body: `미래로당 박선우 후보는 '데이터 기반 맞춤형 복지 서비스 제공'을 통해 "복지 사각지대를 해소하고 효율적인 복지 시스템을 구축하겠습니다"라고 공약했습니다. 이는 복지 효율성을 높이고 필요한 이들에게 정확한 지원을 제공할 수 있다는 점에서 긍정적입니다. 그러나 푸른하늘당 최지아 후보는 "박선우 후보의 정책은 개인 정보 활용에 대한 프라이버시 침해 우려가 큽니다"라고 날카롭게 지적했습니다. 최 후보는 "국민의 기본권 보호를 최우선으로 해야 하며, 보편적 복지 확대를 통해 모두에게 평등한 기회를 제공해야 합니다"라고 주장하며 데이터 활용의 윤리적 측면을 강조했습니다.`,
                source: "오늘의 진실" // 보수 언론사: 중립적/비판적
            },

            // 윤나래 후보 기사 (민주당 성향)
            {
                id: "article-yoonnarae-1",
                candidateName: "윤나래",
                title: "윤나래, '포용적 성장'으로 모두가 잘 사는 사회 구현",
                subtitle: "공정 경제와 주 4일 근무제 검토, 삶의 질 향상 기대",
                body: `함께해요당 윤나래 후보는 '공정 경제 실현'과 '주 4일 근무제 도입 검토'를 통해 노동자의 삶의 질 향상과 사회적 균형 발전을 강조했습니다. 이는 대기업과 중소기업의 상생, 그리고 일과 삶의 균형을 중시하는 포용적 성장을 지향합니다. 윤 후보는 "사회적 대화를 통해 단계적으로 추진하겠습니다"라고 밝혔습니다. 취약 계층을 위한 사회 안전망 강화에도 힘쓰겠다고 강조했습니다. 그녀의 정책은 약자와 소수자를 배려하며 함께 잘 사는 사회를 만들려는 진정성 있는 노력으로 평가받습니다.`,
                source: "미래 탐험 보도" // 진보 언론사: 옹호적
            },
            {
                id: "article-yoonnarae-2",
                candidateName: "윤나래",
                title: "윤나래 후보의 '공공 주택 확대', 시장 왜곡 우려",
                subtitle: "주거 복지 강화 속 부동산 시장의 자율성 침해 논란",
                body: `함께해요당 윤나래 후보는 '주거 복지 확대 및 공공 주택 공급 증대'를 통해 "집 때문에 고통받는 사람이 없도록 하겠습니다"라고 공약했습니다. 이는 주거권을 보장하려는 진보적 접근으로 평가받습니다. 그러나 내일당 강한울 후보는 "윤나래 후보의 정책은 정부의 과도한 개입으로 민간 주택 시장의 활력을 저해하고 시장 왜곡을 초래할 수 있습니다"라고 강하게 비판했습니다. 강 후보는 "민간 주택 공급 활성화와 시장 원리에 따른 집값 안정이 우선되어야 합니다"라고 주장하며 정부 개입의 최소화를 강조했습니다.`,
                source: "오늘의 진실" // 보수 언론사: 비판적
            },
            // 사회적 파장 사건 기사 1: 대규모 전세 사기 사건
            {
                id: "social-issue-1",
                candidateName: "주요 후보들",
                title: "대규모 전세 사기 사건, 후보들의 해법은?",
                subtitle: "청년층 주거 불안 가중, 각 후보의 입장과 대책 발표",
                body: `최근 전국을 뒤흔든 대규모 전세 사기 사건으로 청년층의 주거 불안이 극에 달하고 있습니다. 수많은 피해자가 발생하며 사회적 공분이 커지는 가운데, 대선 후보들은 이 문제에 대한 각기 다른 해법을 제시하며 민심 잡기에 나섰습니다.

기호 1번 강한울 (내일당): 강 후보는 "전세 사기범에 대한 엄벌과 함께 시장의 자율성을 존중해야 한다"고 강조했습니다. 그는 "개인의 재산권 보호와 시장의 건전성 회복이 중요하며, 정부의 과도한 개입은 또 다른 부작용을 낳을 수 있다"고 주장했습니다. 사기 예방을 위한 정보 투명화와 법적 처벌 강화에 집중하겠다는 입장입니다.

기호 2번 최지아 (푸른하늘당): 최 후보는 "전세 사기 피해자 구제를 최우선으로 하고, 공공의 역할을 확대해야 한다"고 목소리를 높였습니다. "집은 투기의 대상이 아닌 주거의 권리이며, 정부가 나서서 임차인을 적극적으로 보호해야 합니다"라고 말하며, 공공 주택 공급 확대와 임차인 보호를 위한 특별법 제정을 공약했습니다.

기호 3번 박선우 (미래로당): 박 후보는 "첨단 기술을 활용하여 주거 정보를 투명하게 공개하고, 데이터 기반으로 사기를 예방해야 한다"고 밝혔습니다. 그는 "블록체인 기반의 부동산 거래 시스템 도입을 통해 사기 발생 가능성을 원천 차단하고, 투명하고 안전한 주거 환경을 조성하겠습니다"라고 기술적 해결책을 제시했습니다.

기호 4번 윤나래 (함께해요당): 윤 후보는 "전세 사기 피해자를 위한 신속하고 실질적인 구제 방안을 마련하고, 공공 지원을 대폭 확대해야 한다"고 강조했습니다. "피해자들의 고통을 외면하지 않고, 정부가 직접 나서서 피해 복구를 돕고 재발 방지 대책을 철저히 수립하겠습니다"라고 말하며, 전세 보증금 반환 보증 강화와 피해자 법률 지원을 약속했습니다.`,
                source: "국민일보"
            },
            // 사회적 파장 사건 기사 2: 기록적인 폭염과 전력난
            {
                id: "social-issue-2",
                candidateName: "주요 후보들",
                title: "기록적 폭염 속 전력난 비상, 에너지 정책 논쟁 가열",
                subtitle: "기후 변화와 에너지 안보, 후보들의 해법은?",
                body: `역대급 폭염이 이어지며 전력 사용량이 급증, 전국적으로 전력난 경보가 발령되는 초유의 사태가 발생했습니다. 기후 변화의 심각성을 다시 한번 일깨우는 이번 사태를 계기로, 대선 후보들의 에너지 정책에 대한 논쟁이 뜨겁습니다.

기호 1번 강한울 (내일당): 강 후보는 "에너지 안보를 최우선으로 확보해야 한다"며 원자력 발전의 중요성을 강조했습니다. 그는 "안정적인 전력 공급을 위해 원전 비중을 확대하고, 기업의 에너지 비용 부담을 완화하여 경제 활력을 되찾아야 한다"고 주장했습니다. 급진적인 재생에너지 전환보다는 현실적인 에너지 믹스를 강조하는 입장입니다.

기호 2번 최지아 (푸른하늘당): 최 후보는 "기후 위기 대응을 위한 재생에너지 전환을 가속화해야 한다"고 목소리를 높였습니다. "이번 전력난은 기후 변화의 경고이며, 에너지 절약 캠페인과 함께 탈탄소 사회로의 전환을 더 빠르게 추진해야 합니다"라고 말하며, 재생에너지 투자 확대와 에너지 효율 향상 정책을 공약했습니다.

기호 3번 박선우 (미래로당): 박 후보는 "첨단 기술을 활용한 에너지 효율 극대화와 차세대 에너지 개발이 시급하다"고 밝혔습니다. 그는 "스마트 그리드 시스템을 도입하여 전력 사용을 효율적으로 관리하고, 수소 에너지 등 미래 에너지 기술 개발에 전폭적인 투자를 통해 에너지 자립을 이루겠습니다"라고 기술 중심의 해결책을 제시했습니다.

기호 4번 윤나래 (함께해요당): 윤 후보는 "기후 위기 시대에 에너지 전환은 선택이 아닌 필수"라며 탈탄소 사회로의 전환을 강조했습니다. "에너지 취약 계층에 대한 지원을 강화하고, 시민들이 직접 참여하는 에너지 정책을 통해 지속 가능한 에너지 시스템을 구축하겠습니다"라고 말하며, 공공 주도의 재생에너지 보급 확대와 에너지 복지를 약속했습니다.`,
                source: "환경일보"
            },
            // 사회적 파장 사건 기사 3: 역대 최저 출생률 발표
            {
                id: "social-issue-3",
                candidateName: "주요 후보들",
                title: "역대 최저 출생률 충격, 대선 후보들 '인구 위기' 해법은?",
                subtitle: "국가 소멸 위기론 확산, 저출생 대책에 관심 집중",
                body: `통계청 발표에 따르면 지난해 합계출산율이 역대 최저치를 기록하며 국가 소멸 위기론이 다시 확산되고 있습니다. 심각한 저출생 문제 해결을 위해 대선 후보들은 각자의 철학을 담은 육아 및 출산 지원 정책을 쏟아내며 유권자들의 표심을 공략하고 있습니다.

기호 1번 강한울 (내일당): 강 후보는 "경제 성장을 통해 양질의 일자리를 창출하고, 기업이 자율적으로 육아 친화적 환경을 조성하도록 지원해야 한다"고 강조했습니다. 그는 "육아 휴직 장려금 확대와 세제 혜택을 통해 기업의 부담을 줄여주고, 경제 활력으로 출산율을 자연스럽게 높이겠습니다"라고 밝혔습니다. 정부의 직접 개입보다는 시장의 역할을 중시하는 입장입니다.

기호 2번 최지아 (푸른하늘당): 최 후보는 "모든 아이는 국가가 함께 키워야 한다"며 보편적 아동 수당 대폭 확대와 공공 보육 시설 확충을 주장했습니다. "육아 휴직을 의무화하고, 남성 육아 참여를 적극 독려하여 성평등한 육아 환경을 조성하겠습니다"라고 말하며, 국가의 책임 있는 돌봄을 강조했습니다.

기호 3번 박선우 (미래로당): 박 후보는 "첨단 기술을 활용하여 육아 부담을 줄이고, 유연 근무를 확산하여 일과 육아의 균형을 찾아야 한다"고 밝혔습니다. 그는 "AI 기반의 맞춤형 육아 정보 플랫폼을 구축하고, 스마트 돌봄 서비스를 도입하여 부모들의 육아 부담을 실질적으로 경감하겠습니다"라고 기술 중심의 해결책을 제시했습니다.

기호 4번 윤나래 (함께해요당): 윤 후보는 "국공립 어린이집을 대폭 확충하고, 육아 휴직 급여를 인상하여 부모들이 안심하고 아이를 키울 수 있는 환경을 만들겠습니다"라고 강조했습니다. "지역 사회 중심의 돌봄 공동체를 활성화하고, 아이를 키우는 모든 가정이 행복할 수 있도록 국가가 적극적으로 지원하겠습니다"라고 말하며, 포괄적인 육아 지원 정책을 약속했습니다.`,
                source: "육아신문"
            }
        ];

        const mainPosterScreen = document.getElementById('main-poster-screen');
        const contentArea = document.getElementById('content-area');
        const contentAreaInner = document.querySelector('#content-area > div'); // Select the inner div
        const navButtons = document.querySelectorAll('.nav-item');
        const candidatePosterCards = document.querySelectorAll('.candidate-poster-card');
        const root = document.documentElement; // HTML root element for CSS variables
        const transitionOverlay = document.getElementById('transition-overlay');
        const transitionMessage = document.getElementById('transition-message');

        // CSS 변수를 업데이트하는 함수
        function updateCssVariables(colors) {
            root.style.setProperty('--candidate-primary-color', colors.primary);
            root.style.setProperty('--candidate-secondary-color', colors.secondary);
            root.style.setProperty('--candidate-background-color', colors.background);
            root.style.setProperty('--candidate-text-color', colors.text);
            root.style.setProperty('--candidate-card-bg-color', colors.cardBg);
            document.body.style.backgroundColor = colors.background;
            document.body.style.color = colors.text;
        }

        // 공약 또는 기사를 렌더링하는 헬퍼 함수
        function renderCards(items, type) {
            let html = '';
            items.forEach(item => {
                html += `
                    <div class="card pledge-card mb-4">
                        <div class="card-header" data-id="${item.id || item.title}">
                            <h3 class="text-xl font-semibold">${item.icon ? item.icon + ' ' : ''}${item.title}</h3>
                            <span class="text-2xl transform transition-transform duration-300">&#x25BC;</span>
                        </div>
                        <div class="card-content">
                            ${type === 'pledge' ?
                                item.details.map(detail => `<p class="mb-2">${detail.replace(/\*\*(.*?)\*\*/g, '$1')}</p>`).join('') : // Remove ** **
                                `<p class="font-medium mb-2">${item.subtitle}</p>
                                <p class="text-gray-700 whitespace-pre-wrap">${item.body}</p>
                                <p class="text-sm text-right mt-4 text-gray-500">출처: ${item.source}</p>`
                            }
                        </div>
                    </div>
                `;
            });
            return html;
        }

        // 후보자 프로필 및 공약 페이지 렌더링
        function renderCandidatePage(candidateKey) {
            const candidate = candidatesData[candidateKey];
            if (!candidate) return;

            updateCssVariables(candidate.colors); // Update CSS variables for candidate colors

            let html = `
                <header class="py-8 text-center shadow-md header-bg rounded-lg mb-8">
                    <h1 class="text-4xl font-bold header-text-color">${candidate.name} 후보</h1>
                    <p class="text-xl mt-1 header-text-color">${candidate.party}</p>
                    <p class="text-2xl font-semibold mt-4 italic px-4 header-text-color">"${candidate.slogan}"</p>
                </header>

                <section id="about" class="py-10">
                    <h2 class="text-3xl font-bold text-center mb-6">후보자 소개</h2>
                    <div class="p-6 rounded-lg shadow-lg text-lg leading-relaxed bio-bg w-full">
                        <p class="mb-4">${candidate.bio}</p>
                        <h3 class="text-2xl font-bold mb-3">주요 경력</h3>
                        <p>${candidate.career}</p>
                    </div>
                </section>

                <nav class="sticky top-0 z-40 shadow-md nav-bg mt-8 rounded-lg">
                    <div class="max-w-7xl mx-auto px-2 sm:px-4 lg:px-6">
                        <div class="flex flex-wrap items-center justify-center py-2 gap-2">
                            <div class="flex flex-wrap justify-center gap-2" id="pledge-nav-buttons">
                                <button data-category="all" class="nav-item active px-3 py-2 rounded-md text-sm font-medium">✨ 전체 공약</button>
                            </div>
                        </div>
                    </div>
                </nav>

                <main id="pledges" class="py-10">
                    <h2 class="text-3xl font-bold text-center mb-8">후보자 공약</h2>
                    <div id="pledge-list" class="space-y-6">
                        </div>
                </main>

                <footer class="py-12 text-center footer-bg rounded-lg mt-8">
                    <p class="text-2xl font-semibold footer-text-color">"${candidate.slogan.replace('!', '')}을 위해, ${candidate.name}이(가) 함께 하겠습니다!</p>
                    <p class="text-2xl font-semibold mt-2 footer-text-color">여러분의 소중한 한 표 부탁드립니다!"</p>
                    <p class="text-sm mt-6 footer-text-color">본 공보물은 대화형 웹 애플리케이션으로 제작되었습니다.</p>
                </footer>
            `;
            contentAreaInner.innerHTML = html; /* Insert into inner div */
            
            // Explicitly set display styles
            mainPosterScreen.style.display = 'none';
            contentArea.style.display = 'flex'; // Ensure contentArea is a flex container when visible

            // Update active state in top navigation
            navButtons.forEach(btn => btn.classList.remove('active'));
            document.getElementById(`nav-${candidateKey}`).classList.add('active');


            // 공약 카테고리 버튼 동적 생성
            const pledgeNavButtonsContainer = document.getElementById('pledge-nav-buttons');
            const categories = [...new Set(candidate.pledges.map(p => p.category))];
            let categoryButtonsHtml = '<button data-category="all" class="nav-item active px-3 py-2 rounded-md text-sm font-medium">✨ 전체 공약</button>';
            const categoryMap = {
                "security": "🛡️ 안보 & 외교",
                "economy": "📈 경제 & 일자리",
                "welfare": "🤝 복지 & 건강",
                "culture": "🎓 교육 & 문화",
                "environment": "🌎 환경 & 안전",
                "public": "🏛️ 공공 효율",
                "justice": "⚖️ 법치 & 정의",
                "agriculture": "🍎 농수산",
                "gender": "🚻 성평등",
                "social": "🌱 사회적 경제",
                "animal": "🐾 동물 복지",
                "human_rights": "🤝 인권", /* Changed icon to be less specific */
                "politics": "🗳️ 정치 개혁",
                "innovation": "💡 혁신 성장",
                "governance": "📱 디지털 정부",
                "space": "🌌 우주 산업",
                "global": "🌍 글로벌 협력",
                "youth": "🧑‍🎓 청년",
                "finance": "💰 재정",
                "peace": "🕊️ 평화 & 통일",
                "region": "🌆 지역 균형",
                "health": "🏥 보건 의료",
                "transport": "🚌 교통",
                "safety_net": "🛡️ 사회 안전망",
                "digital_inclusion": "📱 디지털 포용"
            };
            categories.forEach(cat => {
                categoryButtonsHtml += `<button data-category="${cat}" class="nav-item px-3 py-2 rounded-md text-sm font-medium">${categoryMap[cat] || cat}</button>`;
            });
            pledgeNavButtonsContainer.innerHTML = categoryButtonsHtml;

            const pledgeListContainer = document.getElementById('pledge-list');
            const pledgeNavButtons = document.querySelectorAll('#pledge-nav-buttons .nav-item');

            // 공약 표시 함수
            function displayPledges(category = 'all') {
                pledgeListContainer.innerHTML = '';
                const filteredPledges = category === 'all'
                    ? candidate.pledges
                    : candidate.pledges.filter(pledge => pledge.category === category);

                if (filteredPledges.length === 0) {
                    pledgeListContainer.innerHTML = `<p class="text-center text-lg">선택하신 분야의 공약이 없습니다.</p>`;
                    return;
                }
                pledgeListContainer.innerHTML = renderCards(filteredPledges, 'pledge');
                addCardToggleListeners(pledgeListContainer);
            }

            // 공약 카테고리 버튼 이벤트 리스너
            pledgeNavButtons.forEach(button => {
                button.addEventListener('click', () => {
                    pledgeNavButtons.forEach(btn => btn.classList.remove('active'));
                    button.classList.add('active');
                    displayPledges(button.dataset.category);
                });
            });

            displayPledges(); // 초기 공약 로드
        }

        // 신문 기사 페이지 렌더링
        function renderNewsPage() {
            // Reset CSS variables to a neutral theme for news page
            updateCssVariables({
                primary: "#4A5568", /* Gray-700 */
                secondary: "#2D3748", /* Gray-800 */
                background: "#F8F8F8", /* Light Gray */
                text: "#333333", /* Dark Gray */
                cardBg: "#FFFFFF" /* White */
            });

            let html = `
                <header class="py-8 text-center shadow-md bg-gray-700 text-white rounded-lg mb-8">
                    <h1 class="text-4xl font-bold">📰 신문 기사</h1>
                    <p class="text-xl mt-1">후보자 공약 검증 및 분석</p>
                </header>

                <main id="news-articles" class="py-10">
                    <h2 class="text-3xl font-bold text-center mb-8">주요 언론사 기사</h2>
                    <div id="article-list" class="space-y-6">
                        ${newspaperArticles.map(article => `
                            <div class="card pledge-card mb-4">
                                <div class="card-header" data-id="${article.id}">
                                    <h3 class="text-xl font-semibold">${article.candidateName === "주요 후보들" ? "" : "후보: "}${article.title}</h3>
                                    <span class="text-2xl transform transition-transform duration-300">&#x25BC;</span>
                                </div>
                                <div class="card-content">
                                    <p class="font-medium mb-2">${article.subtitle}</p>
                                    <p class="text-gray-700 whitespace-pre-wrap">${article.body}</p>
                                    <p class="text-sm text-right mt-4 text-gray-500">출처: ${article.source}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                </main>
            `;
            contentAreaInner.innerHTML = html; /* Insert into inner div */
            mainPosterScreen.style.display = 'none';
            contentArea.style.display = 'flex';
            addCardToggleListeners(document.getElementById('article-list'));

            // Update active state in top navigation
            navButtons.forEach(btn => btn.classList.remove('active'));
            document.getElementById('nav-news').classList.add('active');
        }

        // 밸런스 게임 페이지 렌더링
        function renderBalanceGamePage() {
            updateCssVariables({
                primary: "#CFD8DC", /* Blue Gray 100 - Lighter for header */
                secondary: "#78909C", /* Blue Gray 400 */
                background: "#ECEFF1", /* Light Blue Gray */
                text: "#263238", /* Dark Blue Gray */
                cardBg: "#FFFFFF" /* White */
            });

            let html = `
                <header class="py-8 text-center shadow-md header-bg text-gray-800 rounded-lg mb-8">
                    <h1 class="text-4xl font-bold">⚖️ 밸런스 게임</h1>
                </header>

                <main id="balance-game-main" class="balance-game-container">
                    <button id="start-balance-game" class="start-button">게임 시작</button>
                    <div id="question-area" class="hidden w-full flex flex-col items-center">
                        <p id="question-progress" class="text-lg font-medium mb-4"></p>
                        <div id="question-card" class="question-card">
                            <h2 id="question-text" class="text-2xl font-bold mb-8"></h2>
                            <div class="flex flex-col space-y-4">
                                <button id="option-a" class="option-button"></button>
                                <button id="option-b" class="option-button"></button>
                            </div>
                        </div>
                    </div>
                    <div id="result-area" class="hidden result-card">
                        <h2 class="text-3xl font-bold mb-4">당신의 선택 요약</h2>
                        <p id="result-text" class="text-xl text-gray-700 mb-6 whitespace-pre-wrap"></p>
                        <button id="restart-balance-game" class="start-button">다시 하기</button>
                    </div>
                </main>
            `;
            contentAreaInner.innerHTML = html; /* Insert into inner div */
            mainPosterScreen.style.display = 'none';
            contentArea.style.display = 'flex';

            // Update active state in top navigation
            navButtons.forEach(btn => btn.classList.remove('active'));
            document.getElementById('nav-balance-game').classList.add('active');

            setupBalanceGame();
        }

        // 밸런스 게임 로직
        const balanceGameQuestions = [
            {
                question: "경제 정책의 방향은? 💰",
                options: [
                    { text: "정부 주도의 분배와 복지 확대 🤝", value: 1, summary: "정부 주도의 분배와 복지 확대를 선호합니다." }, // 진보
                    { text: "시장 중심의 성장과 규제 완화 🚀", value: -1, summary: "시장 중심의 성장과 규제 완화를 선호합니다." } // 보수
                ]
            },
            {
                question: "사회적 불평등 해소를 위해 중요한 것은? ⚖️",
                options: [
                    { text: "적극적인 정부 개입과 재분배 💖", value: 1, summary: "적극적인 정부 개입과 재분배를 중요하게 생각합니다." },
                    { text: "개인의 노력과 자유로운 경쟁 💪", value: -1, summary: "개인의 노력과 자유로운 경쟁을 중요하게 생각합니다." }
                ]
            },
            {
                question: "환경 보호와 경제 성장 중 우선순위는? 🌳",
                options: [
                    { text: "환경 보호를 위한 강력한 규제 🌎", value: 1, summary: "환경 보호를 위한 강력한 규제를 우선합니다." },
                    { text: "경제 성장을 위한 규제 완화 🏭", value: -1, summary: "경제 성장을 위한 규제 완화를 우선합니다." }
                ]
            },
            {
                question: "교육의 목표는 무엇이 되어야 하는가? 🎓",
                options: [
                    { text: "보편적 평등 교육과 공공성 강화 🏫", value: 1, summary: "보편적 평등 교육과 공공성 강화를 교육 목표로 봅니다." },
                    { text: "자율과 경쟁을 통한 수월성 추구 🏆", value: -1, summary: "자율과 경쟁을 통한 수월성 추구를 교육 목표로 봅니다." }
                ]
            },
            {
                question: "남북 관계 개선을 위한 접근 방식은? 🕊️",
                options: [
                    { text: "대화와 협력을 통한 평화적 해결 🤝", value: 1, summary: "대화와 협력을 통한 평화적 해결을 선호합니다." },
                    { text: "강력한 안보를 통한 북한 변화 유도 🛡️", value: -1, summary: "강력한 안보를 통한 북한 변화 유도를 선호합니다." }
                ]
            },
            {
                question: "노동 시장의 유연성과 안정성 중 강조할 것은? 🧑‍🏭",
                options: [
                    { text: "노동자의 권리 보호와 고용 안정 💼", value: 1, summary: "노동자의 권리 보호와 고용 안정을 강조합니다." },
                    { text: "기업의 자유로운 경영과 생산성 향상 📊", value: -1, summary: "기업의 자유로운 경영과 생산성 향상을 강조합니다." }
                ]
            },
            {
                question: "국방력 강화의 주요 목적은? 🛡️",
                options: [
                    { text: "평화 유지를 위한 최소한의 방어력 ☮️", value: 1, summary: "평화 유지를 위한 최소한의 방어력을 중요하게 생각합니다." },
                    { text: "외부 위협에 대한 강력한 억지력 🚀", value: -1, summary: "외부 위협에 대한 강력한 억지력을 중요하게 생각합니다." }
                ]
            },
            {
                question: "주택 문제 해결을 위한 최선의 방법은? 🏠",
                options: [
                    { text: "공공 주택 공급 확대 및 투기 규제 🏘️", value: 1, summary: "공공 주택 공급 확대 및 투기 규제를 최선으로 봅니다." },
                    { text: "민간 주택 공급 활성화 및 시장 자율성 보장 🏗️", value: -1, summary: "민간 주택 공급 활성화 및 시장 자율성 보장을 최선으로 봅니다." }
                ]
            }
        ];

        let currentQuestionIndex = 0;
        let userChoices = []; // Store selected options for summary
        let selectedOptionButton = null;

        function setupBalanceGame() {
            const startButton = document.getElementById('start-balance-game');
            const questionArea = document.getElementById('question-area');
            const resultArea = document.getElementById('result-area');
            const restartButton = document.getElementById('restart-balance-game');
            const optionAButton = document.getElementById('option-a');
            const optionBButton = document.getElementById('option-b');

            startButton.onclick = startGame;
            restartButton.onclick = startGame;
            optionAButton.onclick = () => selectOption(0);
            optionBButton.onclick = () => selectOption(1);

            function startGame() {
                currentQuestionIndex = 0;
                userChoices = []; // Reset choices
                selectedOptionButton = null;
                startButton.classList.add('hidden');
                resultArea.classList.add('hidden');
                questionArea.classList.remove('hidden');
                displayQuestion();
            }

            function displayQuestion() {
                if (currentQuestionIndex < balanceGameQuestions.length) {
                    const question = balanceGameQuestions[currentQuestionIndex];
                    document.getElementById('question-progress').textContent = `질문 ${currentQuestionIndex + 1} / ${balanceGameQuestions.length}`;
                    document.getElementById('question-text').textContent = question.question;
                    document.getElementById('option-a').textContent = question.options[0].text;
                    document.getElementById('option-b').textContent = question.options[1].text;

                    // Reset selected state
                    if (selectedOptionButton) {
                        selectedOptionButton.classList.remove('selected');
                    }
                    selectedOptionButton = null;
                } else {
                    showResult();
                }
            }

            function selectOption(optionIndex) {
                const question = balanceGameQuestions[currentQuestionIndex];
                userChoices.push(question.options[optionIndex].summary); // Store the summary text

                // Visually indicate selected option
                if (selectedOptionButton) {
                    selectedOptionButton.classList.remove('selected');
                }
                selectedOptionButton = document.getElementById(optionIndex === 0 ? 'option-a' : 'option-b');
                selectedOptionButton.classList.add('selected');

                // Automatically move to next question after a short delay
                setTimeout(() => {
                    currentQuestionIndex++;
                    displayQuestion();
                }, 500); // 0.5 second delay
            }

            function showResult() {
                questionArea.classList.add('hidden');
                resultArea.classList.remove('hidden');

                let summaryText = "✨ 당신의 선택 요약입니다! ✨\n\n";
                userChoices.forEach((choice, index) => {
                    summaryText += `👉 ${index + 1}. ${choice}\n`;
                });
                summaryText += "\n\n이 선택들이 당신의 가치관을 잘 반영하는지 살펴보세요! 😊\n\n";
                summaryText += "어떤 후보와 가장 가깝다고 생각하시나요? 🤔";

                document.getElementById('result-text').textContent = summaryText;
            }
        }

        // 카드 토글 기능 추가
        function addCardToggleListeners(container) {
            container.querySelectorAll('.card-header').forEach(header => {
                header.addEventListener('click', () => {
                    const content = header.nextElementSibling;
                    const isOpen = content.classList.toggle('open');
                    // Arrow icon rotation is handled by CSS based on .open class
                    if (isOpen) {
                        setTimeout(() => {
                            const cardRect = header.parentElement.getBoundingClientRect();
                            const isPartiallyVisible = cardRect.bottom > window.innerHeight || cardRect.top < 0;
                            if (isPartiallyVisible && (cardRect.bottom > (window.innerHeight *0.8) || cardRect.top < (window.innerHeight*0.2)) ) {
                                header.parentElement.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                            }
                        }, 10);
                    }
                });
            });
        }

        // 메인 포스터 화면 렌더링 함수 추가
        function renderMainPosterScreen() {
            // Reset CSS variables to a neutral theme for the main screen
            updateCssVariables({
                primary: "#4CAF50", /* Default Green */
                secondary: "#429946", /* Default Darker Green */
                background: "#F8F8F8", /* Light Gray */
                text: "#333333", /* Dark Gray */
                cardBg: "#FFFFFF" /* White */
            });
            // Explicitly set display styles
            mainPosterScreen.style.display = 'flex'; // Ensure mainPosterScreen is a flex container when visible
            contentArea.style.display = 'none';

            // Update active state in top navigation
            navButtons.forEach(btn => btn.classList.remove('active'));
            document.getElementById('nav-main').classList.add('active');
        }

        // 메인 네비게이션 버튼 이벤트 리스너
        navButtons.forEach(button => {
            button.addEventListener('click', () => {
                const view = button.dataset.view;
                if (view === 'main') {
                    renderMainPosterScreen();
                } else if (view === 'news') {
                    renderNewsPage();
                } else if (view === 'balance-game') {
                    renderBalanceGamePage();
                }
                else {
                    // Handle candidate page transition with delay
                    const candidateName = candidatesData[view].name;
                    transitionMessage.innerHTML = `${candidateName} 후보 공보물<br>이동중...`;
                    transitionOverlay.classList.add('active');

                    setTimeout(() => {
                        renderCandidatePage(view);
                        transitionOverlay.classList.remove('active');
                    }, 3000); // 3 seconds delay
                }
            });
        });

        // 후보자 포스터 카드 클릭 시 해당 후보 페이지로 이동
        candidatePosterCards.forEach(card => {
            card.addEventListener('click', () => {
                const candidateKey = card.dataset.candidateKey;
                const candidateName = candidatesData[candidateKey].name;

                transitionMessage.innerHTML = `${candidateName} 후보 공보물<br>이동중...`;
                transitionOverlay.classList.add('active');

                setTimeout(() => {
                    renderCandidatePage(candidateKey);
                    transitionOverlay.classList.remove('active');
                }, 3000); // 3 seconds delay
            });
        });

        // 초기 페이지 로드 (메인 포스터 화면을 기본으로 표시)
        document.addEventListener('DOMContentLoaded', () => {
            renderMainPosterScreen();
        });
    </script>

</body>
</html>
