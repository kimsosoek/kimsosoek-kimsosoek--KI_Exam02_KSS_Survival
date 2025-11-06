
3D 서바이벌 슈터 (프로토타입)
제한 시간 180초 동안 몰려드는 AI 적들을 처치하고 최고 점수(Kill Count)를 획득해야 하는 3D TPS(3인칭 슈터) 서바이벌 게임 프로토타입입니다.

이 프로젝트는 언리얼 엔진 5.4 버전을 사용하여 100% 블루프린트로 제작되었습니다.

1. 게임 개요 (Overview)
장르: 3D TPS (3인칭 슈터) 서바이벌

플랫폼: PC (Windows)

플레이 방식: 싱글 플레이

엔진: Unreal Engine 5.4 (Blueprints)

게임 규칙:

180초의 제한 시간 동안 생존합니다.

맵에 주기적으로 스폰되는 적(근접/원거리)을 처치하여 '킬 카운트(점수)'를 올립니다.

주기적으로 스폰되는 MP 물약을 획득하여 스킬을 사용합니다.

플레이어 HP가 0이 되면 '게임 오버' UI가, 시간이 0이 되면 '점수판' UI가 나타납니다.

2. 핵심 기능 상세
2-1. 플레이어 (BP_PlayerCharacter)
플레이어는 이동, 전투, 특수 능력을 갖춘 3D 캐릭터입니다.

입력 시스템: Enhanced Input (인핸스드 인풋) 시스템을 기반으로 WASD 이동, 점프, 마우스 조준(줌) 기능을 구현했습니다.

전투 (사격):

LineTrace (라인 트레이스)를 카메라에서 발사하여 정확한 조준점을 확보합니다.

BP_Projectile_Bullet (총알) 액터를 MuzzleFlashSocket (총구 소켓) 위치에 스폰하고 ProjectileMovement로 발사합니다.

특수 능력:

대시 (Shift): Launch Character를 이용한 회피 기동. Sphere Overlap Actors로 적을 감지합니다.

미사일 (E): BP_Grenade (수류탄) 액터를 스폰하며 MP를 소모합니다. 미사일은 충돌 후 Apply Radial Damage (광역 데미지)를 줍니다.

데미지 및 죽음:

Event AnyDamage (또는 TakeDamage 커스텀 이벤트)를 통해 데미지를 받아 CurrentHP를 갱신합니다.

CurrentHP가 0이 되면, Disable Input (입력 차단), Set Collision Enabled (NoCollision) (충돌 끔), Play Anim Montage (죽음 애니메이션)가 순차적으로 실행된 후, WBP_GameOver (실패 UI)를 띄우고 마우스 커서를 활성화합니다.

2-2. AI 시스템 (BT_Enemy, BP_Enemy_Ranged)
플레이어를 탐지하고 공격하는 2종류의 AI를 구현했습니다.

AI 두뇌 (AI Controller & Behavior Tree):

AIC_Enemy (AI 컨트롤러)는 Event Receive Possess 시 BT_Enemy (비헤이비어 트리)를 실행합니다.

Auto Possess AI 설정은 Placed in World or Spawned (배치 또는 스폰 시)로 설정하여, 맵에 미리 배치된 적(원본)과 게임 모드에 의해 스폰된 적(사본) 모두 AI가 정상 작동하도록 보장했습니다.

AI 탐지 (BTS_FindTarget 서비스):

BI_Searchable (인터페이스) 및 PawnSensing (컴포넌트)를 활용, 0.5초마다 플레이어를 탐지합니다.

플레이어를 찾으면 TargetActor 블랙보드 키에 저장합니다.

AI 행동 분기 (Behavior Tree Logic):

속도 조절: BTT_SetSpeed (커스텀 태스크)를 제작, '추적 상태'일 때는 600(뛰기), '순찰 상태'일 때는 150(걷기)으로 Max Walk Speed를 동적 변경합니다.

공격/이동 분기: Selector (선택자)를 사용하여 "가까우면 공격, 멀면 이동" 로직을 구현했습니다.

공격 (Sequence): BTD_IsCloserThan (커스텀 데코레이터)로 거리를 체크합니다.

이동 (Task): Move To 태스크로 Acceptable Radius (허용 반경)까지 접근합니다.

AI 유형 (2종):

근접 적: BTD_IsCloserThan (150) ➡️ BTT_Attack (광역 데미지).

원거리 적: BTD_IsCloserThan (2000) ➡️ BTT_RangedAttack (총알 발사).

AI 죽음 로직 (버그 해결):

'벌떡' 버그: Event AnyDamage에서 죽음 몽타주 재생 시, ABP_Enemy (애님 BP)의 상태 머신이 Idle로 강제 전환하며 몽타주를 중단시키는 문제.

해결: Event AnyDamage에서 Stop Logic (AI 정지), Disable Movement (움직임 정지)를 먼저 호출하여 애니메이션 덮어쓰기를 방지했습니다. 몽타주가 On Completed되면 Set Simulate Physics (래그돌)로 전환하고 Set Life Span (5초)으로 시체를 자동 제거합니다.

2-3. 애니메이션 시스템 (ABP_Player, ABP_Enemy)
상태 머신 (State Machine): bIsInAir (공중 여부) 변수를 기준으로 '지상'(GroundLocomotion)과 '공중'(AirLocomotion) 상태를 분리하여 관리합니다.

블렌드 스페이스 (Blend Space): GroundSpeed 값에 따라 Idle ↔ Walk ↔ Run 애니메이션이 자연스럽게 블렌딩됩니다.

조준 오프셋 (Aim Offset): ABP_Player에서 Aim Offset을 적용, 이동과 관계없이 마우스(카메라) 방향으로 상체가 회전하며 조준합니다.

몽타주 및 노티파이 (Montage & Notify):

DefaultSlot을 통해 '발사', '대시', '공격', '죽음' 등 각종 스킬/이벤트 애니메이션이 기본 상태를 덮어쓰고 재생됩니다.

ABP_Enemy_Ranged는 AnimNotify_FireRangedBullet 이벤트를 받아, 몽타주 재생 중 정확한 타이밍에 MuzzleFlashSocket에서 총알(BP_Projectile_Bullet1)을 스폰합니다.

2-4. 게임 시스템 및 UI
게임 모드 (GM_SurvivalMode):

게임 타이머: 180초 제한 시간 (UpdateGameTimer 함수).

킬 카운트 (점수): 적 사망 시(Event AnyDamage) GM_SurvivalMode의 KillCount 변수를 +1 증가시킵니다.

자동 스폰: Set Timer를 사용, K2_GetRandomLocationInNavigableRadius로 적(근/원거리 50% 랜덤) 및 MP 물약(BP_MP_Pickup)을 주기적으로 스폰합니다.

UI 및 게임 흐름 제어 (PC_Survival):

메인 메뉴: MAP_MainMenu (기본 맵)에서 WBP_MainMenu 위젯을 띄우고 Set Input Mode UIOnly (UI 모드)로 설정합니다.

게임 시작: '시작' 버튼 클릭 시 MAP_Survival (게임 맵)을 엽니다.

입력 전환: PC_Survival의 Event BeginPlay에서 Get Current Level Name을 체크, MAP_Survival 맵에서는 Add Mapping Context (입력 활성화) 및 Set Input Mode Game Only (게임 모드)로 자동 전환합니다.

HUD: WBP_MainHUD가 HP/MP/부스터/시간/킬 카운트를 실시간 바인딩하여 표시합니다.

게임 종료: 플레이어 사망 시 WBP_GameOver (실패 UI), 시간 종료 시 WBP_Scoreboard (점수판 UI)를 각각 분리하여 표시하고, '다시 시작' / '종료' 버튼을 제공합니다.
