⭐️ UNICON 2024 팀 프로젝트에서 기여했던 것을 구분하기 쉽게 분리한 레포 입니다. [[UNICON 2024 출품작](https://unicon2024.notion.site/UNICON-2024-2efd0ae05b2948eda41b8ba445bb1cde)]

<br>

![히랜디 썸넬](https://github.com/user-attachments/assets/954d0cbd-87d3-4ec1-bb77-978ac028478c)

<br>

✨ [팀 프로젝트 레포지토리](https://github.com/gdevhun/HeroRandomDefence)
<br>
✨ [팀 프로젝트 스케줄링](https://unmarred-deer-17b.notion.site/9198758e37df48ce9535f7564afe2214?pvs=4)

<br>

# 시연 영상  
+ [히랜디](https://youtu.be/7rK0P3Eyj0w)

<br>

# 빌드 파일  
+ [itch.io](https://wjh9330.itch.io/herorandomdefence) : UNICON2024 입력

<br>

# 주요 기능
⚡ 유닛 관리 시스템 [[UnitHandle](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/tree/main/UnitHandle)]
- 유닛 관리 UML
  - GetUnitBase : 모든 유닛 얻는 스크립트의 공통 멤버 정의
  - IConsumable : 재화와 상호작용 하는 스크립트의 공통 멤버 정의
  - GetUnit Implement : GetUnitBase 상속, 필요 시 IConsumable을 다중상속해서 구체화
- ![히랜디 유닛관리 drawio](https://github.com/user-attachments/assets/6d6b37ac-109f-4b79-9df6-9c0275e47f4d)

<br>
<br>

- 유닛 관리 기능
  - GetUnitBase : 모든 유닛 얻는 스크립트의 공통 멤버 정의 [[GetUnitBase](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Base/GetUnitBase.cs)]
  ```csharp
  public abstract class GetUnitBase : MonoBehaviour
  {
      // 유닛이 소환 될 위치 오브젝트들 저장
      // 유닛이 소환되면 랜덤 한 개 위치 오브젝트의 자식으로 설정
      static protected ListGameObject spawnPosList;

      // 특정 유닛을 키로 해서 해당 유닛이 소환된 위치를 맵핑
      // Key : 특정 유닛 타입
      // Value : 유닛이 소환된 위치와 해당 위치의 자식 수
      static public Dictionary<UnitType, Dictionary<GameObject, int> > unitPosMap;
      ....
  
      // 유닛 소환
      public virtual GameObject GetUnit(Dictionary<HeroGradeType, int> gradeWeightMap)
      {
          ....

          // 소환 할 유닛 오브젝트 풀링
          return GetUnitFromPool(gradeWeightMap.ElementAt(i).Key);
      }
  
      // 소환 할 유닛 오브젝트 풀링
      protected virtual GameObject GetUnitFromPool(HeroGradeType heroGradeType)
      {
          return PoolManager.instance.GetPool(PoolManager.instance.unitPool.queMap, (UnitType)UnityEngine.Random.Range(0 + 5 * (int)heroGradeType, 5 + 5 * (int)heroGradeType));
      }
  
      // 유닛 소환 위치 반환
      public virtual GameObject GetUnitPos(UnitType unitType)
      {
          // Case 1.같은 유닛이 스폰된 위치들 중 유닛이 3 개 보다 작은 위치가 있는 경우
          // 해당 위치 반환 => 탐색하면서 자식 수 체크
  
          // 같은 유닛이 스폰된 위치가 있으면서
          if(unitPosMap.ContainsKey(unitType))
          {
              // 같은 유닛이 스폰된 위치들 중
              for(int i = 0; i < unitPosMap[unitType].Count; i++)
              {
                  // 자식 수가 3 개 보다 작을 때 해당 스폰 위치 반환
                  if(unitPosMap[unitType].ElementAt(i).Value >= 3) continue;
                  GameObject unitPos = unitPosMap[unitType].ElementAt(i).Key;
                  unitPosMap[unitType][unitPos]++;
                  return unitPos;
              }
          }
  
          // Case 2. 같은 유닛이 스폰된 위치가 있지만 유닛이 모두 3 개인 경우
          // Case 3. 같은 유닛이 스폰된 위치가 없는 경우
          // 새로운 유닛 스폰 위치 반환 => 아직 유닛이 소환되지 않은 위치들 중 랜덤 스폰 위치 반환
  
          // 아직 유닛이 소환되지 않은 위치들 구하기
          ListGameObject newUnitPosList = new ListGameObject();
          for(int i = 0; i < spawnPosList.gameObjectList.Count; i++)
          {
              if(spawnPosList.gameObjectList[i].transform.childCount > 0) continue;
              newUnitPosList.gameObjectList.Add(spawnPosList.gameObjectList[i]);
          }
  
          // 아직 유닛이 소환되지 않은 위치가 있으면 랜덤 한 개의 새로운 스폰 위치 반환
          if(newUnitPosList.gameObjectList.Count > 0)
          {
              GameObject newUnitPos = newUnitPosList.gameObjectList[UnityEngine.Random.Range(0, newUnitPosList.gameObjectList.Count)];
              unitPosMap[unitType][newUnitPos] = 1;
              return newUnitPos;
          }
  
          // Case 4.스폰되지 않은 새로운 스폰 위치도 없는 경우 => 모든 스폰 위치가 사용 중
          // 소환 가능한 위치 없음
  
          return null;
      }

      ....
  
      // 일반 소환, 도박 소환, 유닛 합성, 신화 조합에서 상속받아 각 기능 구체화
      public abstract void GetUnitHandle();
  }
  ```

  <br>

  - IConsumable : 재화와 상호작용 하는 스크립트의 공통 멤버 정의 [[IConsumable](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Base/IConsumable.cs)]
  ```csharp
  public interface IConsumable
  {
      // 재화 소모 수치
      int amount { get; set; }

      // 재화 소모
      bool ConsumeCurrency();
  }
  ```

  <br>

  - SpawnUnit : 일반 소환 [[SpawnUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Implement/SpawnUnit.cs)]
  - ![일반 소환](https://github.com/user-attachments/assets/26c651bb-81f7-40f9-b094-86bc79e06d0a)
  ```csharp
  public class SpawnUnit : GetUnitBase, IConsumable
  {
      // 1. 일반 소환 등급별 확률 가중치 설정
      public Dictionary<HeroGradeType, int> gradeWeightMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.일반, 72 },
          { HeroGradeType.고급, 24 },
          { HeroGradeType.희귀, 6 },
          { HeroGradeType.전설, 1 }
      };
  
      // 2. 일반 소환 구체화
      public override void GetUnitHandle()
      {
          // 최대 유닛 체크
          if(CurUnit >= maxUnit) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
          
          // 재화 체크
          if(!ConsumeCurrency()) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
          ++SpawnCnt;
  
          // 소환 유닛 구하기
          GameObject instantUnit = GetUnit(gradeWeightMap);
  
          // 소환 위치 구하기
          GameObject unitPos = GetUnitPos(instantUnit.GetComponent<CharacterBase>().heroInfo.unitType);
  
          // 소환 위치가 있는지 체크
          if(unitPos == null) { .... return; }
  
          // 소환 유닛 소환 위치로 이동
          instantUnit.transform.SetParent(unitPos.transform);
          instantUnit.transform.localPosition = new Vector3(unitPos.transform.childCount == 3 ? 0.1f : 0.2f * (unitPos.transform.childCount - 1), unitPos.transform.childCount == 3 ? 0 : 0.2f, -0.1f * (unitPos.transform.childCount - 1));
          ++CurUnit;

          ....
      }
  
      // 3. 재화 구체화

      // 소환 횟수 => 프로퍼티로 설정해서 소모 재화 증가 및 UI 업데이트
      private int spawnCnt;
      public int SpawnCnt
      {
          get { return spawnCnt; } 
          set
          {
              spawnCnt = value;
              amount = 10 + spawnCnt * 3;
              UpdateSpawnGoldUI();
          }
      }

      // 재화 소모 수치
      public int amount { get; set; }

      // 재화 소모
      public bool ConsumeCurrency() { return CurrencyManager.instance.ConsumeCurrency(amount, true); }

      // 재화 UI 업데이트
      [Header ("소환 골드 텍스트")] [SerializeField] private TextMeshProUGUI spawnGoldText;
      private void UpdateSpawnGoldUI() { spawnGoldText.text = amount.ToString(); }
  }
  ```

  <br>

  - GambleUnit : 도박 소환 [[GambleUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Implement/GambleUnit.cs)]
  - ![도박 소환](https://github.com/user-attachments/assets/944684d7-8104-4bdf-bea7-a8b4fad10236)
  ```csharp
  public class GambleUnit : GetUnitBase, IConsumable
  {
      // 1. 도박 소환 등급별 확률 가중치 설정
      private Dictionary<HeroGradeType, int> gradeWeightMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.일반, 50 },
          { HeroGradeType.희귀, 40 },
          { HeroGradeType.전설, 10 }
      };

      ....
  
      // 2. 도박 소환 구체화
      public override void GetUnitHandle()
      {
          // 최대 유닛 체크
          if(CurUnit >= maxUnit) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 재화 체크
          if(!ConsumeCurrency()) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 소환 유닛 구하기
          GameObject instantUnit = GetUnit(gradeWeightMap);
  
          // 소환 위치 구하기 => 소환 유닛이 일반등급이 아닐 때만 가져오기
          GameObject unitPos = null;
          if(instantUnit.GetComponent<CharacterBase>().heroInfo.heroGradeType != HeroGradeType.일반) unitPos = GetUnitPos(instantUnit.GetComponent<CharacterBase>().heroInfo.unitType);
  
          // 소환 위치가 있는지 체크, 일반등급이 아닌지 체크
          if(unitPos == null) { .... return; }
  
          // 소환 유닛 소환 위치로 이동
          instantUnit.transform.SetParent(unitPos.transform);
          instantUnit.transform.localPosition = new Vector3(unitPos.transform.childCount == 3 ? 0.1f : 0.2f * (unitPos.transform.childCount - 1), unitPos.transform.childCount == 3 ? 0 : 0.2f, -0.1f * (unitPos.transform.childCount - 1));
          ++CurUnit;

          ....
      }
  
      // 3. 재화 구체화

      // 재화 소모 수치
      public int amount { get; set; }

      // 재화 소모
      public bool ConsumeCurrency() { return CurrencyManager.instance.ConsumeCurrency(amount, false); }
  }
  ```

  <br>

  - CombUnit : 유닛 합성 [[CombUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Implement/CombUnit.cs)]
  - ![유닛 합성](https://github.com/user-attachments/assets/1180c18a-ca8a-4e8e-b88e-08299076aaab)
  ```csharp
  public class CombUnit : GetUnitBase
  {
      // 1. 유닛 합성 등급별 확률 가중치 설정
      private Dictionary<HeroGradeType, int> NormalCombMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.고급, 100 }
      };
      private Dictionary<HeroGradeType, int> EliteCombMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.일반, 25 },
          { HeroGradeType.희귀, 75 }
      };
      private Dictionary<HeroGradeType, int> RareCombMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.일반, 50 },
          { HeroGradeType.전설, 50 }
      };
  
      // 2. 유닛 합성 구체화
      public override void GetUnitHandle()
      {
          // 유닛이 3 개 인지 체크
          if(SelectUnit.instance.selectedPos.transform.childCount < 3) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 합성 할 유닛에 대한 처리
          // 1. 등급 가져오기
          CharacterBase selectedCharacterBase = SelectUnit.instance.selectedPos.transform.GetChild(0).GetComponent<CharacterBase>();
          HeroGradeType selectedGradeType = selectedCharacterBase.heroInfo.heroGradeType;

          // 전설과 신화는 합성 못 함
          if(selectedGradeType == HeroGradeType.전설 || selectedGradeType == HeroGradeType.신화) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 2. 맵핑 삭제하기
          UnitType selectedUnitType = selectedCharacterBase.heroInfo.unitType;
          unitPosMap[selectedUnitType].Remove(SelectUnit.instance.selectedPos);

          // 3. 부모 해제하고 풀에 반환하기
          for(int i = 0; i < 3; i++)
          {
              GameObject selectedCharacter = SelectUnit.instance.selectedPos.transform.GetChild(0).gameObject;
              selectedCharacter.transform.SetParent(PoolManager.instance.poolSet.transform);
              PoolManager.instance.ReturnPool(PoolManager.instance.unitPool.queMap, selectedCharacter, selectedUnitType);
          }
  
          // 유닛 합성 => 합성 할 유닛의 등급에 따라 합성
          GameObject instantUnit = null;
          switch(selectedGradeType)
          {
              case HeroGradeType.일반 : instantUnit = GetUnit(NormalCombMap); break;
              case HeroGradeType.고급 : instantUnit = GetUnit(EliteCombMap); break;
              case HeroGradeType.희귀 : instantUnit = GetUnit(RareCombMap); break;
          }
  
          // 소환 위치 구하기 => 소환 유닛이 일반등급이 아닐 때만 가져오기
          GameObject unitPos = null;
          if(instantUnit.GetComponent<CharacterBase>().heroInfo.heroGradeType != HeroGradeType.일반) unitPos = GetUnitPos(instantUnit.GetComponent<CharacterBase>().heroInfo.unitType);
  
          ....
  
          // 소환 위치가 있는지 체크, 일반등급이 아닌지 체크
          if(unitPos == null) { .... return; }
  
          // 소환 유닛 소환 위치로 이동
          instantUnit.transform.SetParent(unitPos.transform);
          instantUnit.transform.localPosition = new Vector3(unitPos.transform.childCount == 3 ? 0.1f : 0.2f * (unitPos.transform.childCount - 1), unitPos.transform.childCount == 3 ? 0 : 0.2f, -0.1f * (unitPos.transform.childCount - 1));
          CurUnit -= 2;

          ....
      }
  }
  ```

  <br>

  - MythicUnit : 신화 조합 [[MythicUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/GetUnit/Implement/MythicUnit.cs)]
  - ![신화 조합](https://github.com/user-attachments/assets/87c335c4-9b04-412d-87bf-fe40ae9d9db8)
  ```csharp
  
  // 1. 유저 클래스

  // 신화 조합식
  [Serializable]
  public class MythicComb
  { 
      [Header ("조합 할 신화 유닛")] public UnitType mythicType;
      [Header ("조합 할 신화 유닛 스프라이트")] public Sprite mythicSprite;
      [Header ("필요한 유닛과 수")] public List<UnitRequire> requireList;
  }

  // 각 신화를 조합하는 데 필요한 유닛과 수
  [Serializable]
  public class UnitRequire
  {
      [Header ("필요한 유닛 타입")] public UnitType unitType;
      [Header ("필요한 유닛 등급")] public HeroGradeType gradeType;
      [Header ("필요한 유닛 스프라이트")] public Sprite unitSprite;
      [Header ("필요한 유닛 수")] public int cnt; // 유닛 수 => 같은 유닛 3 개 필요 시 1, 2, 3 할당 => ex) Hunter 3 개 필요 시 Hunter 1, Hunter 2, Hunter 3
  }
  
  public class MythicUnit : GetUnitBase
  {
      // 2. 신화 조합 필드
  
      // 각 신화의 조합식 저장
      [Header ("신화 조합식")] public List<MythicComb> mythicCombList = new List<MythicComb>();

      // 특정 신화 타입에 따른 조합식 맵핑
      // Key : 특정 신화 타입
      // Value : 해당 신화의 조합식
      public Dictionary<UnitType, MythicComb> mythicCombMap = new Dictionary<UnitType, MythicComb>();

      // 현재 선택된 신화 => 프로퍼티로 설정해서 UI 업데이트
      private UnitType selectedMythic;
      public UnitType SelectedMythic
      {
          get { return selectedMythic; }
          set { selectedMythic = value; UpdateMythicInfoUI(value); } 
      }

      ....

      // 3. 신화 조합에 맞게 GetUnitBase 함수 오버라이딩
  
      // 신화 소환
      public override GameObject GetUnit(Dictionary<HeroGradeType, int> gradeWeightMap) { return GetUnitFromPool(HeroGradeType.신화); }
  
      // 소환 할 신화 오브젝트 풀링
      protected override GameObject GetUnitFromPool(HeroGradeType heroGradeType) { return PoolManager.instance.GetPool(PoolManager.instance.unitPool.queMap, selectedMythic); }
  
      // 소환 할 신화 위치 반환
      public override GameObject GetUnitPos(UnitType unitType)
      {
          // Case 1.신화는 한 개의 위치에 한 개만 소환 될 수 있음
          // 새로운 유닛 스폰 위치 반환 => 아직 유닛이 소환되지 않은 위치들 중 랜덤 스폰 위치 반환
  
          // 아직 유닛이 소환되지 않은 위치들 구하기
          ListGameObject newUnitPosList = new ListGameObject();
          for(int i = 0; i < spawnPosList.gameObjectList.Count; i++)
          {
              if(spawnPosList.gameObjectList[i].transform.childCount > 0) continue;
              newUnitPosList.gameObjectList.Add(spawnPosList.gameObjectList[i]);
          }
  
          // 아직 유닛이 소환되지 않은 위치가 있으면 랜덤 한 개의 새로운 스폰 위치 반환
          if(newUnitPosList.gameObjectList.Count > 0)
          {
              GameObject newUnitPos = newUnitPosList.gameObjectList[UnityEngine.Random.Range(0, newUnitPosList.gameObjectList.Count)];
              unitPosMap[unitType][newUnitPos] = 1;
              return newUnitPos;
          }
  
          // Case 2.스폰되지 않은 새로운 스폰 위치도 없는 경우 => 모든 스폰 위치가 사용 중
          // 소환 가능한 위치 없음
  
          return null;
      }

      // 4. 신화 조합에 추가적으로 필요한 함수
  
      // 소환 할 신화 선택 => 프로퍼티 set => 신화 조합 UI 업데이트
      public void SelectMythic(string mythicName) { SelectedMythic = (UnitType)Enum.Parse(typeof(UnitType), mythicName, true); }
  
      // 신화 조합 가능 개수 표시 => 버튼에 연결하기 위해 래핑
      public void WrapCheckMythicComb() { CheckMythicComb(); }
      public int CheckMythicComb()
      {
          // 신화 조합 가능 개수
          int cnt = 0;

          // 각 신화 조합식을 탐색하면서 필요한 유닛이 모두 있으면 카운팅
          for(int i = 4; i > -1; i--)
          {
              SelectMythic(mythicCombList[i].mythicType.ToString());
              
              bool isComb = true;
              for(int j = 0; j < 3; j++)
              {
                  if(!requireImageList[j].transform.GetChild(0).gameObject.activeSelf)
                  {
                      isComb = false;
                      break;
                  }
              }
  
              mythicCombCheckList.gameObjectList[i].SetActive(isComb);
              if(isComb) cnt++;
          }

          // 조합 가능한 신화의 개수 반환
          return cnt;
      }
  
      // 신화 조합 UI 업데이트
      private void UpdateMythicInfoUI(UnitType mythicType)
      {
          mythicImage.sprite = mythicCombMap[mythicType].mythicSprite;
          mythicText.text = mythicType.ToString();
          for(int i = 0; i < requireImageList.Count; i++)
          {
              requireImageList[i].sprite = mythicCombMap[mythicType].requireList[i].unitSprite;
  
              // 유닛 수 구하기
              int unitCnt = 0;
              for(int j = 0; j < unitPosMap[mythicCombMap[mythicType].requireList[i].unitType].Count; j++) unitCnt += unitPosMap[mythicCombMap[mythicType].requireList[i].unitType].ElementAt(j).Key.transform.childCount;
  
              // 보유 유닛 체크 업데이트
              requireImageList[i].transform.GetChild(0).gameObject.SetActive(unitCnt > mythicCombMap[mythicType].requireList[i].cnt - 1);
  
              // 등급별 오라 색 업데이트
              Color newColor = gradeColorMap[mythicCombMap[mythicType].requireList[i].gradeType];
              requireImageList[i].transform.GetChild(1).gameObject.GetComponent<Image>().color = newColor;
          }
      }
  
      // 5. 신화 조합 구체화
      public override void GetUnitHandle()
      {
          // 신화 조합에 사용된 유닛 위치와 수 맵핑
          // Key : 신화 조합에 사용된 유닛의 위치
          // Value : 사용된 유닛의 수
          Dictionary<GameObject, int> usedPosMap = new Dictionary<GameObject, int>();
  
          // 신화 조합에 사용된 유닛 체크
          // Key : 신화 조합에 사용된 유닛 타입
          // Value : 1이면 체크 상태, 키가 없으면 아직 체크하지 않은 상태
          Dictionary<UnitType, int> uesdUnitMap = new Dictionary<UnitType, int>();
  
          // 필요한 유닛과 수가 있는지 체크
          // 1. Hunter가 3 개 필요하면 Hunter 1, Hunter 2, Hunter 3으로 할당됨
          // 2. Hunter가 3마리 인지만 체크 하면 Hunter 1과 Hunter 2는 체크 할 필요 없음
          // 3. 따라서 reverse 체크 => 뒤에 있는 유닛의 cnt 부터 체크
          // 4. 2번에서 설명했듯이 체크 과정에서 이미 체크한 유닛 타입의 경우 continue
          for(int i = mythicCombMap[selectedMythic].requireList.Count - 1; i > -1; i--)
          {
              // 필요한 유닛과 수 가져오기
              UnitRequire require = mythicCombMap[selectedMythic].requireList.ElementAt(i);
  
              // 이미 같은 유닛을 체크한 적이 있는지 체크
              if(uesdUnitMap.ContainsKey(require.unitType)) continue;
              uesdUnitMap.Add(require.unitType, 1);
  
              // 필요한 유닛의 수가 있는지 체크
              bool isCnt = false;
  
              // 필요한 유닛의 위치가 있으면
              if(unitPosMap.ContainsKey(require.unitType))
              {
                  // 존재하는 유닛 수
                  int unitCnt = 0;
  
                  // 필요한 유닛의 위치를 탐색하면서
                  for(int j = 0; j < unitPosMap[require.unitType].Count; j++)
                  {
                      // 필요한 유닛의 위치와 수 가져오기
                      KeyValuePair<GameObject, int> curPos = unitPosMap[require.unitType].ElementAt(j);
                      
                      // 존재하는 유닛 수 누적
                      unitCnt += curPos.Value;
  
                      // 존재하는 유닛 수가 필요한 유닛 수 이상이 되면 break
                      // 조합에 사용된 위치와 초과 하지 않는 유닛 수 맵핑
                      if(unitCnt >= require.cnt)
                      {
                          isCnt = true;
                          usedPosMap[curPos.Key] = require.cnt - (unitCnt - curPos.Value);
                          break;
                      }
  
                      // 아직 더 누적 해야한다면
                      // 조합에 사용된 위치와 자식 수 맵핑
                      usedPosMap[curPos.Key] = curPos.Value;
                  }
  
                  // 필요한 유닛 수가 있으면 다음 필요한 유닛을 체크 하기위해 continue
                  if(isCnt) continue;
              }
              else // 필요한 유닛 수가 충분하지 않은 경우 리턴
              {
                  SoundManager.instance.SFXPlay(SoundType.NotEnough);
                  return;
              }
  
              // 필요한 유닛은 있는데 수가 부족한 경우 리턴
              SoundManager.instance.SFXPlay(SoundType.NotEnough);
              return;
          }
  
          // 신화 소환
          GameObject instantMyth = GetUnit(null);
          GameObject mythPos = GetUnitPos(selectedMythic);
          instantMyth.transform.SetParent(mythPos.transform);
          instantMyth.transform.localPosition = new Vector3(0, 0.2f, 0);
          ++CurUnit;
  
          // 신화 조합에 사용된 유닛 처리
          // 1.자식 수와 사용된 수가 같으면 맵핑 삭제하기, 아니면 자식 수 감소하기
          // 2.부모 해제하고 풀에 반환하기
          // 3.유닛 수 처리
          for(int i = 0; i < usedPosMap.Count; i++)
          {
              // 필요한 유닛의 위치와 수
              KeyValuePair<GameObject, int> curPos = usedPosMap.ElementAt(i);
  
              // 필요한 유닛 타입
              UnitType curUnitType = curPos.Key.transform.GetChild(0).GetComponent<CharacterBase>().heroInfo.unitType;
  
              // 1.자식 수와 사용된 수가 같으면 맵핑 삭제하기, 아니면 자식 수 감소하기
              if(curPos.Key.transform.childCount == curPos.Value) unitPosMap[curUnitType].Remove(curPos.Key);
              else unitPosMap[curUnitType][curPos.Key] -= curPos.Value;
  
              // 2.부모 해제하고 풀에 반환하기
              for(int j = 0; j < curPos.Value; j++)
              {
                  GameObject curUnit = curPos.Key.transform.GetChild(0).gameObject;
                  curUnit.transform.SetParent(PoolManager.instance.poolSet.transform);
                  PoolManager.instance.ReturnPool(PoolManager.instance.unitPool.queMap, curUnit, curUnitType);
              }
  
              // 3.유닛 수 처리
              CurUnit -= curPos.Value;
          }

          ....
      }
  }
  ```

  <br>

  - JackPot : 유닛 잭팟 [[JackPot](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/InteractUnit/JackPot.cs)]
  - ![유닛 잭팟](https://github.com/user-attachments/assets/0e61c2bb-f9d1-4b6c-8202-5e0bfa01d95c)
  ```csharp
  public class JackPot : MonoBehaviour, IConsumable
  {
      // 1. 유닛 잭팟 필드
      [Header ("신화 조합")] [SerializeField] private MythicUnit mythicUnit;
      [Header ("잭팟 유닛이 들어갈 칸")] [SerializeField] private List<Image> jackPotUnitImage = new List<Image>();
      ....

      // 2. 잭팟 핵심 함수
  
      // 잭팟 시작 => 잭팟 버튼에 연결
      public void StartJackPot() { StartCoroutine(JackPotCo()); }
  
      // 잭팟 코루틴
      private IEnumerator JackPotCo()
      {
          // 재화 체크
          if(!ConsumeCurrency()) { SoundManager.instance.SFXPlay(SoundType.NotEnough); yield break; }
  
          // 버튼 비활성화 => 잭팟이 진행 될 동안 다시 누를 수 없게
          jackPotBtn.interactable = false;

          // 1. 잭팟에 사용된 신화 유닛 맵핑
          // Key : 신화 유닛 타입
          // Value : 해당 신화 유닛 수
          Dictionary<UnitType, int> jackPotUnitMap = new Dictionary<UnitType, int>();
          for(int i = 0; i < jackPotUnitImage.Count; i++)
          {
              // 랜덤 신화 유닛 가져오기
              KeyValuePair<UnitType, MythicComb> mythicUnitInfo = GetJackPotUnitInfo();
  
              // 잭팟에 사용된 신화 유닛 맵핑
              if(jackPotUnitMap.ContainsKey(mythicUnitInfo.Key)) jackPotUnitMap[mythicUnitInfo.Key]++;
              else jackPotUnitMap.Add(mythicUnitInfo.Key, 1);
  
              // 잭팟 슬롯 이미지 업데이트
              jackPotUnitImage[i].sprite = mythicUnitInfo.Value.mythicSprite;
              SoundManager.instance.SFXPlay(SoundType.GetUnit);
              yield return StageManager.instance.halfSecond;
          }
  
          // 2. 잭팟 결과 처리
          ResultJackPot(jackPotUnitMap);
      }


      // 3. 잭팟 서브 함수
  
      // 랜덤 신화 유닛 정보 반환
      private KeyValuePair<UnitType, MythicComb> GetJackPotUnitInfo() { return mythicUnit.mythicCombMap.ElementAt(Random.Range(0, mythicUnit.mythicCombMap.Count)); }
  
      // 잭팟 결과 처리
      private void ResultJackPot(Dictionary<UnitType, int> jackPotUnitMap)
      {
          // 버튼 활성화 => 잭팟이 끝나면 다시 시도 할 수 있게
          jackPotBtn.interactable = true;
  
          // 모두 다른 신화 이미지인 경우 => 실패
          if(jackPotUnitMap.Count == 3) { .... return; }
  
          // 같은 신화가 둘인 경우 => 원금 반환
          if(jackPotUnitMap.Count == 2) { CurrencyManager.instance.AcquireCurrency(amount, false); return; }
  
          // 같은 신화가 셋인 경우 => 신화 소환
          GameObject instantMyth = PoolManager.instance.GetPool(PoolManager.instance.unitPool.queMap, jackPotUnitMap.ElementAt(0).Key);
          GameObject mythPos = mythicUnit.GetUnitPos(jackPotUnitMap.ElementAt(0).Key);
          instantMyth.transform.SetParent(mythPos.transform);
          instantMyth.transform.localPosition = new Vector3(0, 0.2f, 0);
          ++GetUnitBase.CurUnit;
      }
  
      // 4. 재화 구체화

      // 재화 소모 수치
      public int amount { get; set; }

      // 재화 소모
      public bool ConsumeCurrency() { return CurrencyManager.instance.ConsumeCurrency(amount, false); }
  }
  ```

  <br>
  <br>

- 유닛 상호작용
  - SelectUnit : 유닛 선택과 이동 [[SelectUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/InteractUnit/SelectUnit.cs)]
  - ![유닛 선택과이동](https://github.com/user-attachments/assets/c7d3dee2-6793-4f0a-aba2-2545db0f9a9d)
  ```csharp
  public class SelectUnit : MonoBehaviour
  {
      // 1. 유닛 선택과 이동 필드
  
      // 선택된 유닛의 위치
      [HideInInspector] public GameObject selectedPos;

      // 유닛의 위치만 클릭되도록 레이어마스크 설정
      [Header ("유닛 스폰 위치만 클릭되게")] [SerializeField] private LayerMask posLayerMask;

      // 선택된 시작 위치 백업
      private Vector3 sPos = Vector3.zero;

      // 클릭과 드래그를 구분하는 임계 값
      private const float dragThreshold = 0.5f;

      // 드래그 상태 체크
      [HideInInspector] public bool isDrag = false;

      // 이동 할 목표 지점 표시
      [Header ("이동 할 위치 표시")] [SerializeField] private GameObject targetPos;

      // 마우스 클릭 시 커서 이미지
      [Header ("마우스 클릭 커서 이미지")] [SerializeField] private Texture2D clickCursorImg;

      // 마우스 입력 체크
      private void Update() { Down(); }

      // 2.유닛 선택과 이동 핵심 함수
  
      // 마우스 클릭
      private void Down()
      {
          // 마우스 클릭 시
          if (Input.GetMouseButtonDown(0) || (Input.touchCount > 0 && Input.GetTouch(0).phase == TouchPhase.Began))
          {
              // 클릭 위치 가져오기
              Vector3 inputPosition = Input.GetMouseButtonDown(0) ? Camera.main.ScreenToWorldPoint(Input.mousePosition) : Camera.main.ScreenToWorldPoint(Input.GetTouch(0).position);
              inputPosition.z = 0;
  
              // 히트된 콜라이더 가져오기
              RaycastHit2D hit = Physics2D.Raycast(inputPosition, Vector2.zero, Mathf.Infinity, posLayerMask);
  
              // 히트된 콜라이더가 있는지 체크, 유닛이 있는지 체크
              if(hit.collider == null || hit.transform.childCount < 1) { .... return; }
  
              // 유닛의 사정거리 표시 끄기
              OnOffIndicateAttackRange(false);
  
              // 선택된 유닛의 위치 저장
              selectedPos = hit.transform.gameObject;
  
              // 선택된 시작 위치 백업
              sPos = selectedPos.transform.position;
  
              // 마우스 커서 변경 및 사운드
              Cursor.SetCursor(clickCursorImg, Vector2.zero, CursorMode.ForceSoftware);
              SoundManager.instance.SFXPlay(SoundType.Click);
          }
  
          // 시작 위치가 있는지 체크
          if(sPos == Vector3.zero) return;
  
          // 마우스 드래그
          Drag();
  
          // 마우스 업
          Up();
      }
  
      // 마우스 드래그
      private void Drag()
      {
          // 마우스를 누르고 있는 동안
          if(Input.GetMouseButton(0) || (Input.touchCount > 0 && Input.GetTouch(0).phase == TouchPhase.Moved))
          {
              // 현재 위치 업데이트
              Vector3 cPos = Input.GetMouseButton(0) ? Camera.main.ScreenToWorldPoint(Input.mousePosition) : Camera.main.ScreenToWorldPoint(Input.GetTouch(0).position);
              cPos.z = 0;
              selectedPos.transform.position = cPos;
              
              // 임계 값을 벗어나면 드래그로 인식
              isDrag = Vector3.Distance(sPos, cPos) > dragThreshold ? true : false;
  
              // 히트된 콜라이더 가져오기
              RaycastHit2D hit = Physics2D.Raycast(selectedPos.transform.position, Vector2.zero, Mathf.Infinity, posLayerMask);
  
              // 이동 할 목표 위치 표시하기
              if(hit.collider != null)
              {
                  targetPos.SetActive(true);
                  targetPos.transform.position = hit.collider == selectedPos.GetComponent<Collider2D>() ? sPos : hit.transform.position;
              }
          }
      }
  
      // 마우스 업
      private void Up()
      {
          // 마우스를 업 했을 때
          if(Input.GetMouseButtonUp(0) || (Input.touchCount > 0 && Input.GetTouch(0).phase == TouchPhase.Ended))
          {
              // 1. 클릭인 경우 => 다시 시작 위치로
              if(!isDrag)
              {
                  // 다시 시작 위치로
                  selectedPos.transform.position = sPos;
  
                  // 시작 위치 초기화
                  sPos = Vector3.zero;
  
                  // 유닛 툴팁 띄우기
                  if(!UiUnit.instance.toolTipPanel.gameObject.activeSelf) UiUnit.instance.OpenPanel(....);
  
                  // 사정거리 표시 켜기
                  OnOffIndicateAttackRange(true);
  
                  // 이동 할 위치 표시 끄기
                  targetPos.SetActive(false);
  
                  // 마우스 커서 이미지 원래대로
                  Cursor.SetCursor(SceneCtrlManager.instance.cursorImg, Vector2.zero, CursorMode.ForceSoftware);
  
                  // 판매 및 합성 패널 띄우기
                  ....
                  UiUnit.instance.OpenPanel(UiUnit.instance.unitSellCompPanel);
                  ....
                  return;
              }
  
              // 2. 드래그인 경우
  
              // 히트된 콜라이더 가져오기
              RaycastHit2D hit = Physics2D.Raycast(selectedPos.transform.position, Vector2.zero, Mathf.Infinity, posLayerMask);
  
              // 2-1. 히트된 콜라이더가 자신 => 경계 벗어난 경우 => 다시 시작 위치로 이동
              if(hit.collider == selectedPos.GetComponent<Collider2D>())
              {
                  // 다시 시작 위치로
                  selectedPos.transform.position = sPos;
  
                  // 시작 위치 초기화
                  sPos = Vector3.zero;
  
                  // 드래그 종료
                  isDrag = false;
  
                  // 이동 할 위치 표시 끄기
                  targetPos.SetActive(false);
  
                  // 마우스 커서 이미지 원래대로
                  Cursor.SetCursor(SceneCtrlManager.instance.cursorImg, Vector2.zero, CursorMode.ForceSoftware);
  
                  return;
              }
  
              // 2-2. 히트된 콜라이더가 있는 경우 => 이동 할 수 있는 위치가 있는 경우 => 목표 위치로 이동
  
              // 목표 위치로
              selectedPos.transform.position = hit.transform.position;
  
              // 목표 위치 오브젝트는 시작 위치로
              hit.transform.position = sPos;
  
              // 시작 위치 초기화
              sPos = Vector3.zero;
  
              // 드래그 종료
              isDrag = false;
  
              // 이동 할 위치 표시 끄기
              targetPos.SetActive(false);
  
              // 마우스 커서 원래대로
              Cursor.SetCursor(SceneCtrlManager.instance.cursorImg, Vector2.zero, CursorMode.ForceSoftware);
          }
      }
      ....
  }
  ```

  <br>

  - UpgradeUnit : 유닛 업그레이드 [[UpgradeUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/InteractUnit/UpgradeUnit.cs)]
  - ![유닛 업그레이드](https://github.com/user-attachments/assets/7abccf32-11dc-4600-8d07-1e6bced91338)
  ```csharp
  public class UpgradeUnit : MonoBehaviour, IConsumable
  {
      // 1. 등급별 업그레이드 수치, 데미지 타입별 업그레이드 수치 관리
      public Dictionary<HeroGradeType, int> gradeUpgradeMap = new Dictionary<HeroGradeType, int>
      {
          { HeroGradeType.일반, 0 },
          { HeroGradeType.고급, 0 },
          { HeroGradeType.희귀, 0 },
          { HeroGradeType.전설, 0 },
          { HeroGradeType.신화, 0 },
      };
      public Dictionary<DamageType, int> damageUpgradeMap = new Dictionary<DamageType, int>
      {
          { DamageType.물리, 0 },
          { DamageType.마법, 0 }
      };

      ....
  
      // 2. 유닛 업그레이드
      private void Upgrade(HeroGradeType heroGradeType)
      {
          // 유닛 등급 백업
          curGradeType = heroGradeType;
  
          // 최대 업그레이드 체크
          int upgradeCnt = curGradeType == HeroGradeType.일반 ? normalUpgradeCnt : legendUpgradeCnt;
          if(upgradeCnt >= 20) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 재화 체크
          amount = curGradeType == HeroGradeType.일반 ? 100 + 25 * upgradeCnt : 2 + upgradeCnt;
          if(!ConsumeCurrency()) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 유닛 업그레이드

          // 1. 반복문 endPoint 설정
          int e = curGradeType == HeroGradeType.일반 ? 3 : 2;

          // 2. 유닛 등급 백업 후 업그레이드 수치 증가
          HeroGradeType standardGradeType = heroGradeType;
          for(int i = 0; i < e; i++) gradeUpgradeMap[standardGradeType++] += 20 + 20 * (upgradeCnt / 5);

          // 3. 업그레이드 횟수 카운팅
          if(curGradeType == HeroGradeType.일반) normalUpgradeCnt++;
          else legendUpgradeCnt++;
  
          // 4. UI 업데이트 및 사운드
          UpdateUpgradeUI(heroGradeType);
          SoundManager.instance.SFXPlay(SoundType.Upgrade);
      }

      // 버튼에 연결 하기위해 래핑
      public void NormalUpgrade() { Upgrade(HeroGradeType.일반); }
      public void LegendUpgrade() { Upgrade(HeroGradeType.전설); }
  
      // 3. 재화 구체화

      // 재화 소모 수치
      public int amount { get; set; }

      // 재화 소모
      public bool ConsumeCurrency()
      {
          if(curGradeType == HeroGradeType.일반) return CurrencyManager.instance.ConsumeCurrency(amount, true);
          return CurrencyManager.instance.ConsumeCurrency(amount, false);
      }

      ....
  }
  ```

  <br>

  - SellUnit : 유닛 판매 [[SellUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/InteractUnit/SellUnit.cs)]
  - ![유닛 판매](https://github.com/user-attachments/assets/29154079-21da-4e68-8d85-b321c1c95090)
  ```csharp
  public class SellUnit : MonoBehaviour
  {
      // 1. 유닛 판매
      public void Sell()
      {
          // 선택된 위치에 유닛이 있는지 체크
          if(SelectUnit.instance.selectedPos.transform.childCount < 1) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 신화 등급은 팔 수 없음
          CharacterBase selectedUnit = SelectUnit.instance.selectedPos.transform.GetChild(0).GetComponent<CharacterBase>();
          HeroGradeType selectedGradeType = selectedUnit.heroInfo.heroGradeType;
          if(selectedGradeType == HeroGradeType.신화) { SoundManager.instance.SFXPlay(SoundType.NotEnough); return; }
  
          // 판매 할 유닛 처리
  
          // 1.유닛이 한 개면 맵핑 삭제하기, 아니면 자식 수 감소하기
          UnitType selectedUnitType = selectedUnit.heroInfo.unitType;
          if(SelectUnit.instance.selectedPos.transform.childCount == 1) GetUnitBase.unitPosMap[selectedUnitType].Remove(SelectUnit.instance.selectedPos);
          else --GetUnitBase.unitPosMap[selectedUnitType][SelectUnit.instance.selectedPos];
  
          // 2.가장 마지막 자식 부모 해제하고 풀에 반환하기
          GameObject selectedCharacter = SelectUnit.instance.selectedPos.transform.GetChild(SelectUnit.instance.selectedPos.transform.childCount - 1).gameObject;
          selectedCharacter.transform.SetParent(PoolManager.instance.poolSet.transform);
          PoolManager.instance.ReturnPool(PoolManager.instance.unitPool.queMap, selectedCharacter, selectedUnitType);
  
          // 3.재화 처리
          if(selectedGradeType == HeroGradeType.일반 || selectedGradeType == HeroGradeType.고급) CurrencyManager.instance.AcquireCurrency(soldierCnt > 0 ? 2 * (20 + 20 * (int)selectedGradeType) : 20 + 20 * (int)selectedGradeType, true);
          else if(selectedGradeType == HeroGradeType.희귀 || selectedGradeType == HeroGradeType.전설) CurrencyManager.instance.AcquireCurrency((int)selectedGradeType, false);
  
          // 4.유닛 수 처리
          GetUnitBase.CurUnit -= 1;

          ....
      }
  }
  ```

  <br>

  - ToolTipUnit : 유닛 툴팁 정보 [[ToolTipUnit](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/UnitHandle/InteractUnit/ToolTipUnit.cs)]
  - ![유닛 툴팁](https://github.com/user-attachments/assets/a158c1a5-e007-4340-828d-677bccd006f7)
  ```csharp
  public class ToolTipUnit : MonoBehaviour
  {
      1. 유닛 툴팁 정보를 표시 할 UI 필드
      [Header ("유닛 이미지")] [SerializeField] private Image unitImage;
      ....
  
      // 선택된 유닛의 툴팁 정보 업데이트
      public void SetToolTip(HeroInfo heroInfo, AbilityUiInfo abilityInfo, AbilityUiInfo hiddenAbilityInfo, CharacterBase characterBase)
      {
          unitImage.sprite = heroInfo.unitSprite;
          unitNameText.text = heroInfo.unitType.ToString();
          unitInfoText.text = $"{heroInfo.heroGradeType} / {heroInfo.damageType} / {heroInfo.attackType}";
          unitDmgText.text = ((int)characterBase.GetApplyAttackDamage(characterBase.heroInfo.attackDamage)).ToString();
          unitSpeedText.text = heroInfo.attackSpeed.ToString();
          abilityImage.sprite = abilityInfo.abilitySprite;
          abilityNameText.text = characterBase.GetComponent<AbilityManage>().maxStamina > 0 ? abilityInfo.abilityName + $" (스태미너 : {characterBase.GetComponent<AbilityManage>().maxStamina})" : abilityInfo.abilityName;
          abilityContentText.text = abilityInfo.abilityContent;
          ....
      }
  }
  ```
<br>
<br>

⚡ 유닛 스킬 시스템 [[Ability](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/tree/main/Ability)]
- 스킬 데이터 : Scriptable Object
- ![히랜디 스킬 스크립터블](https://github.com/user-attachments/assets/2b145cca-c842-4841-b591-51b7bf7cd8ab)

<br>

- 유닛 스킬 UML
  - Ability Base : Scriptable Object 상속 => 모든 스킬 공통 멤버 정의
  - IHidden Ability : 히든 스킬을 갖는 스크립트의 공통 멤버 정의
  - AsyncAbility Base : Ability Base 상속 => 비동기 스킬 공통 멤버 정의
  - SyncAbility Base : Ability Base 상속 => 동기 스킬 공통 멤버 정의
  - Ability Implement : 기반 스킬 상속, 필요 시 히든 스킬을 다중상속해서 각 스킬 구체화
  - Ability Manage : Ability Base 참조 => 각 스킬 데이터가 갖는 상태를 관리
- ![히랜디 스킬](https://github.com/user-attachments/assets/edab0974-cd2d-461f-b966-a4d9b310c2d4)

<br>

- 스킬 시전 메커니즘 [[AbilityManage](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/Ability/Base/AbilityManage.cs)]
- ![유닛 스킬](https://github.com/user-attachments/assets/f795cd1c-2f4b-42c7-b233-14c2dc695171)
  ```csharp
  public class AbilityManage : MonoBehaviour
  {
      // Ability Base 참조 => 다형성을 이용해 구체화 클래스의 스킬이 시전됨
      [Header ("스킬 정보")] public AbilityBase ability;
  
      // 스태미너 => 프로퍼티로 설정해서 스태미너가 가득 차게되면 스킬을 시전함
      private int stamina;
      public int Stamina
      {
          get { return stamina; }
          set
          {
              stamina = value;
  
              // 최대 스태미너가 되고 타겟이 존재 할 때 스킬 시전
              if((louizyCnt == 0 && stamina >= maxStamina && characterBase.isOnTarget) || (louizyCnt > 0 && stamina >= maxStamina - 1 && characterBase.isOnTarget))
              {
                  // 드래그인 경우 => 스킬 시전 안 함
                  if(SelectUnit.instance.isDrag && transform.parent.gameObject.name == SelectUnit.instance.selectedPos.name) return;

                  // 스킬 시전
                  if(ability.abilitySoundType != SoundType.GetUnit) SoundManager.instance.SFXPlay(ability.abilitySoundType);
                  if(ability is SyncAbilityBase syncAbilityBase) syncAbilityBase.CastAbility(characterBase);
                  else if(ability is AsyncAbilityBase asyncAbilityBase) StartCoroutine(asyncAbilityBase.CastAbility(characterBase));
                  stamina = 0;
              }

              // 스태미너 UI 업데이트
              UpdateStaminaUI();
          }
      }

      ....
  
      // 스태미너 증가 코루틴
      private IEnumerator StaminaIncreament()
      {
          while(true)
          {
              ++Stamina;
              yield return StageManager.instance.oneSecond;
          }
      }

      ....
  }
  ```

  <br>
  <br>

⚡ 스테이지 관리 시스템
- Stage Data : Scriptable Object 상속 => 스테이지 공통 멤버 정의
- Stage Manager : Stage Data를 참조해서 관리 => 스테이지 시작 및 종료 관리 [[StageManager](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/Manager/StageManager/StageManager.cs)]
```csharp
public class StageManager : MonoBehaviour
{
    // 1. 스테이지 필드

    // 스테이지 데이터 관리
    [HideInInspector] public List<StageData> stageList = new List<StageData>();

    // 현재 스테이지 => 프로퍼티로 설정 => 스테이지가 끝난 후 재화 획득 및 다음 스테이지 시작, 마지막 스테이지가 끝난 후 게임 클리어
    [Header ("최대 스테이지")] public int maxStage;
    private int curStage;
    public int CurStage
    {
        get { return curStage; }
        set
        {
            curStage = value;
            if(curStage < maxStage)
            {
                // 다음 스테이지 시작
                StartStage(value);
                CurrencyManager.instance.AcquireCurrency((curStage - 1) * 10 + CurrencyManager.instance.Gold / 10, true);
            }
            else
            {
                // 게임 클리어
                GameManager.instance.PlayerGameWin();
                StopAllCoroutines();
            }
        }
    }

    ....

    // 현재 몬스터 수 => 프로퍼티로 설정 => UI 업데이트, 최대 몬스터 수를 넘어가면 게임 실패
    [Header ("최대 몬스터 수")] [SerializeField] private int maxEnemyCnt;
    [HideInInspector] public float maxEnemyFloatCnt;
    private int enemyCnt;
    public int EnemyCnt
    {
        get { return enemyCnt; }
        set
        {
            if(value < 0) return;
            
            enemyCnt = value;
            if(EnemyCnt > maxEnemyCnt)
            {
                // 게임 실패
                GameManager.instance.PlayerGameOver();
                StopAllCoroutines();
            }

            // 몬스터 수 UI 업데이트
            UpdateEnemyCntUI(value);
        }
    }

    ....

    // 2. 스테이지 핵심 함수

    // 스테이지 초기화
    private void InitStage()
    {
        // 1. 몬스터 이동경로 맵핑
        // Key : 스폰 위치
        // Value : 이동 경로
        stageTypePathMap.Add(pathPosList.gameObjectList[0], pathList[0]);
        stageTypePathMap.Add(pathPosList.gameObjectList[1], pathList[1]);
        stageTypePathMap.Add(pathPosList.gameObjectList[2], pathList[2]);

        // 2. 기준 스테이지 몬스터 스탯 맵핑
        // Key : 스테이지 인덱스
        // Value : 기준 스테이지 스탯
        for(int i = 0; i < standardMonsterStatList.Count; i++) standardMonsterStatMap.Add(i, standardMonsterStatList[i]);

        // 3. 스테이지 데이터 생성 및 초기화
        for(int i = 0; i < maxStage; i++)
        {
            // 스테이지 데이터 생성
            StageData stageData = ScriptableObject.CreateInstance<StageData>();

            // 스테이지 번호
            stageData.stageNumber = i + 1;

            // 스테이지 타입
            if(stageData.stageNumber % 10 == 0) stageData.stageType = StageType.Boss;
            else stageData.stageType = stageData.stageNumber % 5 == 0 ? StageType.MiniBoss : StageType.Normal;

            // 스테이지 시간, 몬스터 타입, 몬스터 소환 위치, 스테이지 재화
            switch(stageData.stageType)
            {
                case StageType.Normal :
                    stageData.stageTime = 20;
                    stageData.enemyType = (EnemyType)(stageData.stageNumber / 5 * 2);
                    stageData.spawnPos.gameObjectList.Add(pathPosList.gameObjectList[0]);
                    stageData.spawnPos.gameObjectList.Add(pathPosList.gameObjectList[1]);
                    stageData.stageCurrency = 1 + stageData.stageNumber / 20;
                    break;
                case StageType.MiniBoss :
                    stageData.stageTime = 30;
                    ....
                    break;
                case StageType.Boss :
                    stageData.stageTime = 60;
                    ....
                    break;
            }

            // 스테이지 데이터 리스트에 추가
            stageList.Add(stageData);
        }
    }

    // 스테이지 시작
    public void StartStage(int cur)
    {
        // 스테이지 타입에 따라 스테이지 시작
        StageData stage = stageList[cur];
        switch(stage.stageType)
        {
            // 일반 스테이지 시작
            case StageType.Normal : StartCoroutine(StartNormalStage(stage)); break;

            // 미니보스 스테이지 시작
            case StageType.MiniBoss : StartCoroutine(StartMiniBossStage(stage)); break;

            // 보스 스테이지 시작
            case StageType.Boss : StartCoroutine(StartBossStage(stage)); break;
        }
        
        // UI 업데이트
        UpdateStageNumUI(stage);
        UpdateStageTimeUI(stage.stageTime);
    }

    // 일반 스테이지 시작
    private IEnumerator StartNormalStage(StageData stage)
    {
        // 배경음
        switch(stage.stageNumber)
        {
            case 1 : SoundManager.instance.BgmSoundPlay(BgmType.구간1에서9); break;
            case 11 : SoundManager.instance.BgmSoundPlay(BgmType.구간11에서19); break;
            ....
            default : break;
        }

        // 몬스터 소환 => 스테이지 시간 만큼 1초 마다 소환
        int stageTime = stage.stageTime;
        for(int i = 0; i < stage.stageTime; i++)
        {
            EnemySpawn(stage);
            yield return oneSecond;
            UpdateStageTimeUI(--stageTime);
        }

        // 스테이지 시간이 끝나면 다음 스테이지 => 프로퍼티가 set 되면서 다음 스테이지 시작
        ++CurStage;
    }

    // 미니보스 스테이지 시작
    private IEnumerator StartMiniBossStage(StageData stage)
    {
        // 몬스터 소환 => 미니보스 한 번 소환
        GameObject miniBoss = EnemySpawn(stage);
        int stageTime = stage.stageTime;
        for(int i = 0; i < stage.stageTime; i++)
        {
            yield return oneSecond;

            // 필드에 몬스터가 없으면 시간단축
            if(EnemyCnt == 0 && stageTime > 10)
            {
                i = 0;
                stageTime = 10;
                stage.stageTime = 10;
                UpdateStageTimeUI(10);
                continue;
            }

            UpdateStageTimeUI(--stageTime);
        }

        // 미니보스 잡았는지 체크 => 못 잡은 경우 풀에 반환
        if(miniBoss.activeSelf)
        {
            PoolManager.instance.ReturnPool(PoolManager.instance.enemyPool.queMap, miniBoss, stage.enemyType);
            EnemyCnt--;
        }

        // 스테이지 시간이 끝나면 다음 스테이지 => 프로퍼티가 set 되면서 다음 스테이지 시작
        ++CurStage;
    }

    // 보스 스테이지 시작
    private IEnumerator StartBossStage(StageData stage)
    {
        // 배경음
        switch(stage.stageNumber)
        {
            case 10 : SoundManager.instance.BgmSoundPlay(BgmType.스테이지10); break;
            case 20 : SoundManager.instance.BgmSoundPlay(BgmType.스테이지20); break;
            ....
            default : break;
        }

        // 몬스터 소환 => 보스 한 번 소환
        GameObject boss = EnemySpawn(stage);
        int stageTime = stage.stageTime;
        int tempStageTime = stage.stageTime;
        for(int i = 0; i < stage.stageTime; i++)
        {
            yield return oneSecond;

            // 스테이지 시간 중간에 보스를 잡았으면
            if(!boss.activeSelf)
            {
                // 마지막 스테이지에서 잡았으면 => 게임 클리어
                if(stage.stageNumber == maxStage)
                {
                    GameManager.instance.PlayerGameWin();
                    yield break;
                }

                // 필드에 몬스터가 없으면 시간단축
                if(EnemyCnt == 0 && stageTime > 10)
                {
                    i = 0;
                    stageTime = 10;
                    stage.stageTime = 10;
                    UpdateStageTimeUI(10);
                    continue;
                }
            }

            UpdateStageTimeUI(--stageTime);
            tempStageTime--;
        }

        // 보스 잡았는지 체크 => 못 잡은 경우 게임 실패
        if(boss.activeSelf)
        {
            // 게임 실패
            GameManager.instance.PlayerGameOver();
            yield break;
        }

        // 스테이지 시간이 끝나면 다음 스테이지 => 프로퍼티가 set 되면서 다음 스테이지 시작
        ++CurStage;
    }

    // 몬스터 소환
    private GameObject EnemySpawn(StageData stage)
    {
        // 소환된 몬스터
        GameObject instantEnemy = null;

        // 몬스터 풀링 및 스탯 초기화
        for(int i = 0; i < stage.spawnPos.gameObjectList.Count; i++)
        {
            // 1. 몬스터 풀링
            instantEnemy = PoolManager.instance.GetPool(PoolManager.instance.enemyPool.queMap, stage.enemyType);
            instantEnemy.transform.position = stage.spawnPos.gameObjectList[i].transform.position;
            ....

            // 2. 몬스터 스탯 설정

            // 일반 스테이지
            if(stage.stageType != StageType.MiniBoss && stage.stageType != StageType.Boss)
            {
                enemyBase.maxHp = standardMonsterStatMap[CurStage / 10].hp + standardMonsterStatMap[CurStage / 10].increaseHp * CurStage;
                enemyBase.CurrentHp = standardMonsterStatMap[CurStage / 10].hp + standardMonsterStatMap[CurStage / 10].increaseHp * CurStage;
                ....
                continue;
            }

            // 미니 보스 및 보스 스테이지
            enemyBase.maxHp = stage.stageType == StageType.MiniBoss ? standardMonsterStatMap[CurStage / 10].bossHp : standardMonsterStatMap[CurStage / 10].bossHp * 5;
            enemyBase.CurrentHp = stage.stageType == StageType.MiniBoss ? standardMonsterStatMap[CurStage / 10].bossHp : standardMonsterStatMap[CurStage / 10].bossHp * 5;
            ....
        }

        // 3. 소환된 몬스터 리턴 => 미니보스 및 보스 오브젝트를 가져오도록 리턴
        return instantEnemy;
    }

    ....
}
```

  <br>
  <br>

⚡ 오브젝트 관리 시스템
- Pool Manager : 유닛, 몬스터, 이펙트, 사운드, 플로팅 텍스트 오브젝트 관리 [[PoolManager](https://github.com/LeeJungHwi/HeroRandomDefense-TeamProject/blob/main/Manager/EtcManager/PoolManager.cs)]
```csharp
// 1. 오브젝트 맵핑 클래스 => 제네릭 사용 => T에 따라 다양한 타입의 오브젝트 관리
[Serializable]
public class Pool<T>
{
    [Header ("모든 프리팹")] public List<ListGameObject> allList = new List<ListGameObject>();
    public Dictionary<T, GameObject> prefMap = new Dictionary<T, GameObject>(); // (타입, 프리팹) 맵핑
    public Dictionary<T, Queue<GameObject> > queMap = new Dictionary<T, Queue<GameObject> >(); // (타입, 큐) 맵핑
}

// 2. 오브젝트 타입

// 유닛
public enum UnitType
{
    갱스터, 헌터, 솔져, 시프, 레슬러,
    알렉스, 어쌔신, 바바리안, 벙커, 에키온,
    알론소, 에이든, 뱃, 통키, 바이킹,
    아아솔, 알리스다, 배니스, 루이지, 막더스,
    배트맨, 테오도르, 마그너스, 마리오, 유미
}

// 기본공격 이펙트
public enum WeaponEffect
{
    GansterMelee, HunteBullet, SoldierBullet, ThiefMelee, WrestlerMelee,
    AlexBullet, AssassinMelee, BarbarianMelee, BunkerBullet, EkionMelee,
    AlonsoMelee, AdenMelee, BatBullet, TonkeyBullet, VikingMelee,
    AasoleBullet, AlisdaMelee, BaniesBullet, LouizyBullet, MakdusMelee,
    BatmanBullet, TeodorMelee, MagnusMelee, MarioBullet, YumieMelee,
    BatmanAbilityBullet, MarioAbilityBullet
}

// 사운드
public enum SoundType
{
    GetUnit, Sell, Click, Upgrade, NotEnough, Roulette, MythicComb,
    평타각목둔기, 평타철둔기, 평타핸드, 평타에키온에이든테오도르, 총알마법평타, 총알물리평타,
    갱스터스킬, 레슬러스킬, 시프스킬, 헌터스킬,
    바바리안스킬, 벙커스킬, 알렉스스킬, 어쌔신스킬,
    알론소스킬, 통키스킬,
    막더스스킬, 알리스다스킬,
    배트맨스킬, 마리오스킬, 마그너스스킬, 테오도르스킬, 유미스킬1, 유미스킬2
}

// 몬스터
public enum EnemyType
{
    Normal1, MiniBoss1, Normal2, Boss1,
    Normal3, MiniBoss2, Normal4, Boss2,
    Normal5, MiniBoss3, Normal6, Boss3,
    Normal7, MiniBoss4, Normal8, Boss4,
    Normal9, MiniBoss5, Normal10, Boss5
}

// 스킬 이펙트
public enum AbilityEffectType
{
    스턴, 시프, 통키, 알론소, 알리스다, 막더스,
    배트맨, 마리오, 마그너스, 테오도르, 유미슬로우, 유미빛
}

// 플로팅 텍스트
public enum FloatingTextType
{
    데미지플로팅
}

public class PoolManager : MonoBehaviour
{
    ....

    // 3. 관리 할 타입의 오브젝트 맵핑 클래스 생성
    [Header ("유닛 풀")] public Pool<UnitType> unitPool;
    [Header ("히어로 무기")] public Pool<WeaponEffect> weaponEffectPool;
    [Header ("사운드 풀")] public Pool<SoundType> soundPool;
    [Header ("몬스터 풀")] public Pool<EnemyType> enemyPool;
    [Header ("스킬 이펙트 풀")] public Pool<AbilityEffectType> abilityEffectPool;
    [Header ("플로팅 텍스트 풀")] public Pool<FloatingTextType> floatingTextPool;

    private void Awake()
    {
        // 4. (타입, 프리팹) 맵핑
        PrefMap(unitPool.allList, unitPool.prefMap);
        PrefMap(soundPool.allList, soundPool.prefMap);
        PrefMap(weaponEffectPool.allList,weaponEffectPool.prefMap);
        PrefMap(enemyPool.allList, enemyPool.prefMap);
        PrefMap(abilityEffectPool.allList, abilityEffectPool.prefMap);
        PrefMap(floatingTextPool.allList, floatingTextPool.prefMap);

        // 5. (타입, 큐) 맵핑
        QueMap(unitPool.queMap, unitPool.prefMap);
        QueMap(soundPool.queMap, soundPool.prefMap, 100);
        QueMap(weaponEffectPool.queMap,weaponEffectPool.prefMap, 500);
        QueMap(enemyPool.queMap, enemyPool.prefMap, 130);
        QueMap(abilityEffectPool.queMap, abilityEffectPool.prefMap);
        QueMap(floatingTextPool.queMap, floatingTextPool.prefMap, 20000);
    }

    // 오브젝트 풀링 핵심 함수 => 다양한 타입의 오브젝트를 관리 하기위해 제네릭 T 사용, T의 타입을 Enum으로 제한

    // 1. (타입, 프리팹) 맵핑
    private void PrefMap<T>(List<ListGameObject> pList, Dictionary<T, GameObject> pMap) where T : Enum
    {
        // 타입 초기화
        T curType = default;

        for(int i = 0; i < pList.Count; i++)
        {
            for(int j = 0; j < pList[i].gameObjectList.Count; j++)
            {
                // 사운드 타입 설정 => 재생이 끝나면 설정된 사운트 타입으로 풀에 반환 하기위해
                if(typeof(T) == typeof(SoundType)) pList[i].gameObjectList[j].GetComponent<SoundDeActive>().type = (SoundType)(object)curType;

                // 현재 타입에 해당하는 프리팹 맵핑
                pMap.Add(curType, pList[i].gameObjectList[j]);
                
                // 다음 타입
                curType = (T)(object)(((int)(object)curType) + 1);
            }
        }
    }

    // 2. (타입, 큐) 맵핑
    private void QueMap<T>(Dictionary<T, Queue<GameObject> > qMap, Dictionary<T, GameObject> pMap, int cnt = 50) where T : Enum
    {   
        // 타입을 하나씩 꺼내서
        foreach(T type in Enum.GetValues(typeof(T)))
        {
            // 타입에 해당하는 프리팹을 하나씩 꺼내서
            Queue<GameObject> queue = new Queue<GameObject>();
            GameObject prefab = pMap[type];

            // cnt 개 생성하고 비활성화 후 큐에 저장
            for(int i = 0; i < cnt; i++)
            {
                // 프리팹 생성
                GameObject obj = Instantiate(prefab);

                // 비활성화
                obj.SetActive(false);

                // 큐에 저장
                queue.Enqueue(obj);
            }

            // (타입, 큐) 맵핑
            qMap.Add(type, queue);
        }
    }

    // 3. 풀에서 가져오기
    public GameObject GetPool<T>(Dictionary<T, Queue<GameObject> > qMap, T type) where T : Enum
    {
        // 키가 존재하고 큐에 오브젝트가 있으면 꺼냄
        if(qMap.ContainsKey(type) && qMap[type].Count > 0)
        {
            // 오브젝트 꺼내서
            GameObject obj = qMap[type].Dequeue();

            // 오브젝트 활성화
            obj.SetActive(true);

            // 오브젝트 반환
            return obj;
        }

        // 사용 가능한 오브젝트가 없으면
        return null;
    }

    // 4. 풀에 반환하기
    public void ReturnPool<T>(Dictionary<T, Queue<GameObject> > qMap, GameObject obj, T type) where T : Enum
    {
        // 오브젝트 비활성화
        obj.SetActive(false);

        // 해당 타입의 풀로 반환
        qMap[type].Enqueue(obj);
    }
}

// 사운드 풀링
public class SoundManager : MonoBehaviour
{
    ....
    public void SFXPlay(SoundType type)
    {
        PoolManager.instance.GetPool(PoolManager.instance.soundPool.queMap, type).GetComponent<AudioSource>().volume = sfxVolume;
    }
    ....
}
```

<br>
<br>

⚡ 유닛과 몬스터 상호작용
```csharp

// 1. 공격력에 업그레이드 수치 적용
public float GetApplyAttackDamage(float basicAttackDamage)
{
    float applyAttack = heroInfo.damageType == DamageType.물리
        ? (basicAttackDamage +
           basicAttackDamage * UpgradeUnit.instance.damageUpgradeMap[DamageType.물리] / 100
           + basicAttackDamage * UpgradeUnit.instance.gradeUpgradeMap[heroInfo.heroGradeType] / 100)
        : (basicAttackDamage +
           basicAttackDamage * UpgradeUnit.instance.damageUpgradeMap[DamageType.마법] / 100
           + basicAttackDamage * UpgradeUnit.instance.gradeUpgradeMap[heroInfo.heroGradeType] / 100);
        
    return applyAttack;
}

// 2. 몬스터 피격 데미지에 방어력 수치 적용
public void TakeDamage(float damage, DamageType dmgType)
{
    float applyDmg = damage;

    // 물리 데미지
    if(dmgType == DamageType.물리)
    {
        float applyPhyDef = phyDef - DecreasePhyDef;
        applyDmg = applyPhyDef > 0 ? applyDmg * (1 - phyDef / (phyDef + 100)) : damage;
        CurrentHp -= applyDmg;
        if(applyDmg > 0) FloatingDmg(applyDmg);
        return;
    }

    // 마법 데미지
    float applyMagDef = magDef - decreaseMagDef;
    applyDmg = applyMagDef > 0 ? applyDmg * (1 - magDef / (magDef + 100)) : damage;
    CurrentHp -= applyDmg;
    if(applyDmg > 0) FloatingDmg(applyDmg);
}

// 3. 몬스터 타격
private void OnTriggerEnter2D(Collider2D other)
{
    if (other.gameObject.CompareTag("Enemy"))
    {
        Collider2D[] hits = Physics2D.OverlapCircleAll(transform.position, 0.001f);
        foreach (Collider2D hit in hits)
        {
            if (hit.CompareTag("Enemy"))
            {
                EnemyBase enemyBase = hit.GetComponent<EnemyBase>();

                // 공격력의 업그레이드와 몬스터의 방어력을 적용한 최종 데미지로 몬스터 타격
                enemyBase.TakeDamage(characterBase.GetApplyAttackDamage(attackDamage), damageType);
            }
        }
    }
}

// 4. 플로팅 데미지
private void FloatingDmg(float dmg)
{
    // 플로팅 데미지 풀링
    FloatingText instantFloatingText = PoolManager.instance.GetPool(PoolManager.instance.floatingTextPool.queMap, FloatingTextType.데미지플로팅).GetComponent<FloatingText>();

    // 플로팅 데미지 텍스트 설정, 단위 변환 K & M
    if(dmg < 1000) instantFloatingText.text.text = $"{dmg:0.0}";
    else if(dmg < 1000000) instantFloatingText.text.text = $"{dmg / 1000f:0.0}K";
    else instantFloatingText.text.text = $"{dmg / 1000000f:0.0}M";

    // 플로팅 데미지 위치 설정
    Vector3 worldToScreen = Camera.main.WorldToScreenPoint(transform.position + Vector3.up * 0.5f);
    RectTransformUtility.ScreenPointToLocalPointInRectangle((RectTransform)instantFloatingText.canvasTransform, worldToScreen, null, out Vector2 localPoint);
    instantFloatingText.GetComponent<RectTransform>().localPosition = localPoint;
}

// 5.스턴

// 스턴 시간 설정
private float curStunTime;
public float SetStunTime
{
    get { return curStunTime; }
    set
    {
        // 스턴 시간 설정
        curStunTime = value;
        moveSpeed = 0;
        ....
    }
}

// 스턴 시간이 설정되면 스턴 시작
private void Update() { GetStun(); }
private void GetStun()
{
    if(curStunTime > 0)
    {
        // 스턴시간 감소
        curStunTime -= Time.deltaTime;

        // 스턴시간이 끝나면 스턴 종료
        if(curStunTime <= 0)
        {
            curStunTime = 0;
            moveSpeed = originMoveSpeed;
            ....
        }
    }
}
```
