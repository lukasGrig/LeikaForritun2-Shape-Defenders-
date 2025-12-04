# General

## Player.cs
```csharp
using TMPro;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Player : MonoBehaviour
{

    public static Player main;

    [SerializeField] private int HP = 20;
    public int Money = 100;

    [SerializeField] private GameObject GameOverGUI;
    [SerializeField] private GameObject WinGUI;

    [SerializeField] private TextMeshProUGUI HPGUI;
    [SerializeField] private TextMeshProUGUI MoneyGUI;

    [SerializeField] private AudioClip hurtSound;
    [SerializeField] private AudioClip loseSound;
    [SerializeField] private AudioClip winSound;

    private AudioSource audioSource;

    void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        main = this;
    }
    void Update()
    {
        HPGUI.text = "HP: " + HP.ToString();
        MoneyGUI.text = "Money: "+ Money.ToString();
    }
    public void damage(int damage)
    {
        HP -= damage;

        audioSource.PlayOneShot(hurtSound);

        if (HP <= 0)
        {
            GameOverGUI.SetActive(true);
            GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
            for (int i = 0; i < enemies.Length; i++)
            {
                Destroy(enemies[i]);
            }
            MusicPlayer.main.pauseMusic();
            audioSource.PlayOneShot(loseSound);
        }
    }
    public void Restart()
    {
        string currentStringName = SceneManager.GetActiveScene().name;
        SceneManager.LoadScene(currentStringName);
    }
    public void Finish()
    {
        WinGUI.SetActive(true);
        MusicPlayer.main.pauseMusic();
        audioSource.PlayOneShot(winSound);
    }
}
```

## MusicPlayer.cs
```csharp
using TMPro;
using UnityEngine;

public class MusicPlayer : MonoBehaviour
{
    public static MusicPlayer main;

    private AudioSource audioSource;

    public bool music = true;
    public bool sound = true;

    [SerializeField] private Color green;
    [SerializeField] private Color red;

    [SerializeField] private TextMeshProUGUI musicText;
    [SerializeField] private TextMeshProUGUI soundText;

    [SerializeField] private int index = 0;

    [SerializeField] AudioClip grasslandsMusic;
    [SerializeField] AudioClip desertMusic;
    [SerializeField] AudioClip antarticaMusic;
    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        main = this;

        audioSource.loop = true;

        musicText.color = green;
        soundText.color = green;

        playMusic();
    }

    public void pauseMusic()
    {
        audioSource.Pause();
    }

    public void toggleMusic()
    {
        music = !music;

        if (music)
        {
            musicText.color = green;
            playMusic();
        }
        else
        {
            musicText.color = red;
            audioSource.Stop();
        }
    }
    public void toggleSound()
    {
        sound = !sound;

        if (sound)
        {
            soundText.color = green;
        }
        else
        {
            soundText.color = red;
        }
    }

    public void playMusic()
    {
        switch (index)
        {
            case 0:
                audioSource.clip = grasslandsMusic;
                audioSource.Play();
                break;
            case 1:
                audioSource.clip = desertMusic;
                audioSource.Play();
                break;
            case 2:
                audioSource.clip = antarticaMusic;
                audioSource.Play();
                break;
        }
    }
}

```

## LevelManager.cs
```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class LevelManager : MonoBehaviour
{
    public void loadScene(int index)
    {
        SceneManager.LoadScene(index);
    }
}

```

## PopUp.cs
```csharp
using UnityEngine;

public class PopUp : MonoBehaviour
{

    [SerializeField] private GameObject PopUpUI;

    private void Awake()
    {
        PopUpUI.SetActive(false);
    }

    public void showPopUp()
    {
        PopUpUI.SetActive(true);
    }
    public void unShowPopUp()
    {
        PopUpUI.SetActive(false);
    }
}

```

--------------------------------------------------------------------------------------------------------------------------------------------------

# Tower

## TowerManager.cs
```csharp
using NUnit.Framework;
using TMPro;
using UnityEngine;
using UnityEngine.EventSystems;
using System.Collections.Generic;


public class TowerManager : MonoBehaviour
{
    [Header("Towers")]
    [SerializeField] private GameObject Basic_Tower;
    [SerializeField] private GameObject Bomb_Tower;
    [SerializeField] private GameObject Cannon_Tower;
    [SerializeField] private GameObject Sniper_Tower;
    [SerializeField] private GameObject Lightning_Tower;
    [SerializeField] private GameObject Ice_Tower;
    [SerializeField] private GameObject Music_Tower;


    [SerializeField] private float cashBackPercent = 0.7F;

    [SerializeField] private LayerMask towerLayer;
    [SerializeField] private LayerMask restrictedLayer;


    [SerializeField] private GameObject panel;
    [SerializeField] private TextMeshProUGUI towerName;
    [SerializeField] private TextMeshProUGUI towerLevel;
    [SerializeField] private TextMeshProUGUI UpgradeCost;
    [SerializeField] private TextMeshProUGUI towerTargetting;

    [SerializeField] private TextMeshProUGUI PlacingText;

    private AudioSource audioSource;

    [SerializeField] private AudioClip sellSound;
    [SerializeField] private AudioClip upgradeSound;

    private GameObject selectedTower;
    private GameObject placingTower;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        PlacingText.gameObject.SetActive(false);

    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1))
        {
            setTower(Basic_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha2))
        {
            setTower(Cannon_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha3)){
            setTower(Bomb_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha4))
        {
            setTower(Sniper_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha5))
        {
            setTower(Music_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha6))
        {
            setTower(Ice_Tower);
        }
        else if (Input.GetKeyDown(KeyCode.Alpha7))
        {
            setTower(Lightning_Tower);
        }

        else if (Input.GetKeyDown(KeyCode.E))
        {
            UpgradeSelected();
        }
        else if (Input.GetKeyDown(KeyCode.X))
        {
            SellSelected();
        }
        else if (Input.GetKeyDown(KeyCode.Z))
        {
            ChangeTargetting();
        }
        else if (Input.GetKeyDown(KeyCode.B) || Input.GetKeyDown(KeyCode.Escape))
        {
            clearSelected();
        }

        if (placingTower != null)
        {
            if (!placingTower.GetComponent<TowerPlacement>().isPlacing)
            {
                placingTower = null;
                PlacingText.gameObject.SetActive(false);
            }
            else
            {
                PlacingText.gameObject.SetActive(true);
            }
        }
        if (Input.GetMouseButtonDown(0))
        {
            RaycastHit2D hit = Physics2D.Raycast(Camera.main.ScreenToWorldPoint(Input.mousePosition),Vector2.zero,100f,towerLayer);
            //RaycastHit2D restrictedHit = Physics2D.Raycast(Camera.main.ScreenToWorldPoint(Input.mousePosition), Vector2.zero, 100f, restrictedLayer);

            // && restrictedHit.collider == null
            if (hit.collider != null)
            {
                if ((hit.collider.gameObject.GetComponent<TowerPlacement>().RestrictedAreas.Count <= 0 && Player.main.Money >= hit.collider.gameObject.GetComponent<Tower>().cost) || !placingTower)
                {
                    if (selectedTower)
                    {
                        GameObject range1 = selectedTower.transform.GetChild(1).gameObject;

                        range1.GetComponent<SpriteRenderer>().enabled = false;
                    }

                    selectedTower = hit.collider.gameObject;

                    GameObject range2 = selectedTower.transform.GetChild(1).gameObject;

                    range2.GetComponent<SpriteRenderer>().enabled = true;

                    panel.SetActive(true);
                    towerName.text = selectedTower.name.Replace("(Clone)", "").Trim();

                    towerLevel.text = "LVL: " + selectedTower.GetComponent<TowerUpgrades>().currentLevel.ToString();
                    UpgradeCost.text = selectedTower.GetComponent<TowerUpgrades>().currentCost;

                    Tower tower = selectedTower.GetComponent<Tower>();

                    if (tower.First == true)
                    {
                        towerTargetting.text = "First";
                    }
                    else if (tower.Last == true)
                    {
                        towerTargetting.text = "Last";
                    }
                    else if (tower.Strongest == true)
                    {
                        towerTargetting.text = "Strongest";
                    }
                    else if (tower.Weakest == true)
                    {
                        towerTargetting.text = "Weakest";
                    }
                    else if (tower.Closest == true)
                    {
                        towerTargetting.text = "Closest";
                    }
                }
                
            }
            else if (!EventSystem.current.IsPointerOverGameObject() && selectedTower) // get kannski gera hit.collider = null í staðinn
            {
                panel.SetActive(false);

                GameObject range1 = selectedTower.transform.GetChild(1).gameObject;

                range1.GetComponent<SpriteRenderer>().enabled = false;

                selectedTower = null;
            }
        }
    }

    private void clearSelected()
    {
        if (placingTower)
        {
            Destroy(placingTower);
            placingTower= null;
            PlacingText.gameObject.SetActive(false);
        }
    }

    public void setTower(GameObject tower)
    {
        clearSelected();
        placingTower = Instantiate(tower);
    }

    public void UpgradeSelected()
    {
        if (selectedTower)
        {
            TowerUpgrades Upgrades = selectedTower.GetComponent<TowerUpgrades>();
            
            Upgrades.Upgrade();
            towerLevel.text = "LVL: " + Upgrades.currentLevel.ToString();
            UpgradeCost.text = Upgrades.currentCost;
            if (MusicPlayer.main.sound)
            {
                audioSource.PlayOneShot(upgradeSound);
            }
        }
    }
    public void SellSelected()
    {
        if (selectedTower)
        {
            int moneyBack = SellAmount(); 

            Player.main.Money += Mathf.RoundToInt(moneyBack*cashBackPercent);

            if (selectedTower.GetComponent<Effects>().usingMusic)
            {
                List<GameObject> towers = selectedTower.transform.GetChild(1).GetComponent<TowerRange>().boostedTowers;
                Effects effects = selectedTower.GetComponent<Effects>();

                for (int i = 0; i < towers.Count; i++)
                {
                    effects.UnapplyEffectTower(towers[i]);
                }
            }

            Destroy(selectedTower);

            panel.SetActive(false);
            selectedTower = null;

            if (MusicPlayer.main.sound)
            {
                audioSource.PlayOneShot(sellSound);
            }
        }
    }
    public void ChangeTargetting()
    {
        if (selectedTower)
        {
            Tower tower = selectedTower.GetComponent<Tower>();

            if (tower.First)
            {
                tower.First = false;
                tower.Last = true;
                tower.Strongest = false;
                tower.Weakest = false;
                tower.Closest = false;
                towerTargetting.text = "Last";
            }

            else if (tower.Last)
            {
                tower.First = false;
                tower.Last = false;
                tower.Strongest = true;
                tower.Weakest = false;
                tower.Closest = false;
                towerTargetting.text = "Strongest";
            }

            else if (tower.Strongest)
            {
                tower.First = false;
                tower.Last = false;
                tower.Strongest = false;
                tower.Weakest = true;
                tower.Closest = false;
                towerTargetting.text = "Weakest";
            }

            else if (tower.Weakest)
            {
                tower.First = false;
                tower.Last = false;
                tower.Strongest = false;
                tower.Weakest = false;
                tower.Closest = true;
                towerTargetting.text = "Closest";
            }

            else if (tower.Closest)
            {
                tower.First = true;
                tower.Last = false;
                tower.Strongest = false;
                tower.Weakest = false;
                tower.Closest = false;
                towerTargetting.text = "First";
            } 
        }
    }
    private int SellAmount()
    {
        TowerUpgrades levels = selectedTower.GetComponent<TowerUpgrades>();
        Tower tower = selectedTower.GetComponent<Tower>();

        int moneyBack = tower.cost;

        int currentLevel = levels.currentLevel;

        for (int i = 0; i < currentLevel; i++)
        {
            moneyBack += levels.levels[i].cost;
        }
        return moneyBack;
    }
}
```

## Tower.cs
```csharp
using System;
using System.Collections;
using Unity.VisualScripting;
using UnityEngine;

public class Tower : MonoBehaviour
{

    [Header("Tower stats")]
    public float range = 10f;
    public int damage = 1;
    public float firerate = 1f;
    public int cost = 50;

    [SerializeField] private Effects effects;

    [NonSerialized] public float firerateMultiplier = 0f;
    [NonSerialized] public float stunDuration = 0f;

    [SerializeField] public bool isSupport = false;

    [Header("Targeting mode")]

    public bool First = true;
    public bool Last = false;
    public bool Strongest = false;
    public bool Weakest = false;
    public bool Closest = false; 

    [NonSerialized]public GameObject target;

    private float cooldown = Mathf.Infinity;

    [Header("Misc")]

    [SerializeField] private GameObject stunIndicator;
    [NonSerialized] public SpriteRenderer stunRenderer;

    [SerializeField] private float lineDuration = 0.05f;
    private LineRenderer lineRenderer;

    [SerializeField] private AudioClip clip;
    private AudioSource audiosource;

    private void Start()
    {
        if(stunIndicator != null)
        {
            stunRenderer = stunIndicator.GetComponent<SpriteRenderer>();
            stunRenderer.enabled = false;
        }
        else
        {
            Debug.LogWarning("No stun indicator");
        }
        if(GetComponent<LineRenderer>() != null){
            lineRenderer = GetComponent<LineRenderer>();
            lineRenderer.positionCount = 2;
            lineRenderer.enabled = false;
        }
        if (GetComponent<AudioSource>() != null)
        {
            audiosource = GetComponent<AudioSource>();
        }
    }
    void Update()
    {
        if (!isSupport)
        {
            if (target && stunDuration <= 0f)
            {
                if (cooldown >= (firerate * (1 - firerateMultiplier)))
                {
                    Vector2 dir = (target.transform.position - transform.position);
                    dir.Normalize();

                    float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;
                    transform.rotation = Quaternion.Euler(0f, 0f, angle); // fékk aðstoð frá yt fyrir þetta

                    if (clip != null && MusicPlayer.main.sound)
                    {
                        audiosource.PlayOneShot(clip);
                    }

                    StartCoroutine(spawnLine());

                    target.GetComponent<Enemy>().damageTaken(damage);

                    if (effects != null)
                    {
                        effects.ApplyEffectEnemy(target.gameObject);
                    }

                    cooldown = 0f;
                }
                else 
                {
                    cooldown += Time.deltaTime;
                }
            }
            else
            {
                cooldown += Time.deltaTime;
            }
            if (stunDuration > 0f)
            {
                stunIndicator.transform.Rotate(0, 0, 90f * Time.deltaTime);
                stunDuration -= Time.deltaTime;

                if (stunDuration <= 0f)
                {
                    stunRenderer.enabled = false;
                    stunIndicator.transform.rotation = Quaternion.identity;
                }
            }
        }
    }
    IEnumerator spawnLine()
    {
        lineRenderer.SetPosition(0, gameObject.transform.position);
        lineRenderer.SetPosition(1, target.transform.position);
        lineRenderer.enabled = true;
        yield return new WaitForSeconds(lineDuration);
        lineRenderer.enabled = false;
    }
}
```

## TowerRange.cs
```csharp
using NUnit.Framework;
using UnityEngine;
using System.Collections.Generic;
using Unity.VisualScripting;
using System;

public class TowerRange : MonoBehaviour
{
    [SerializeField] private Tower tower;

    [NonSerialized] public List<GameObject> boostedTowers = new List<GameObject>();
    private List<GameObject> targets = new List<GameObject>();

    private GameObject MainTarget;

    [SerializeField] private Effects effects; 
    void Awake()
    {
        updateRange();
    }

    void Update()
    {
        // targeting 
        if (targets.Count > 0) {
            
            if (tower.First)
            {
                MainTarget = CheckFirst();
            }
            else if (tower.Last)
            {
                MainTarget = CheckLast();
            }
            else if (tower.Strongest)
            {
                MainTarget = CheckStrongest();
            }
            else if (tower.Weakest)
            {
                MainTarget = CheckWeakest();
            }
            else if (tower.Closest)
            {
                MainTarget = CheckClosest();
            }
            else
            {
                MainTarget = targets[0];
            }

            tower.target = MainTarget;    
        }
        else
        {
            tower.target = null;   
        }
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.gameObject.tag == "Enemy")
        {
            targets.Add(collision.gameObject);
        }
        else if(collision.gameObject.tag == "Tower" && tower.isSupport && !boostedTowers.Contains(collision.gameObject))
        {
            GameObject towerBoosted = collision.gameObject;
            boostedTowers.Add(towerBoosted);

            effects.ApplyEffectTower(towerBoosted);
        }
    }
    private void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.gameObject.tag == "Enemy")
        {
            targets.Remove (collision.gameObject);
        }
        else if (collision.gameObject.tag == "Tower" && tower.isSupport && boostedTowers.Contains(collision.gameObject))
        {
            GameObject towerBoosted = collision.gameObject;
            boostedTowers.Remove(towerBoosted);

            effects.UnapplyEffectTower(towerBoosted);
        }
    }
    public void updateRange()
    {
        transform.localScale = new Vector3(tower.range, tower.range, tower.range);
    }
    private GameObject CheckFirst()
    {
        float minDistance = Mathf.Infinity;
        int maxIndex = 0;

        foreach (GameObject target in targets)
        {
            int index = target.GetComponent<Enemy>().index;
            float distance = target.GetComponent<Enemy>().distance;

            if (index > maxIndex || (index == maxIndex && distance < minDistance))
            {
                maxIndex = index;
                minDistance = distance;
                MainTarget = target;
            }
        }

        return MainTarget;
    }
    private GameObject CheckLast()
    {
        float maxDistance = -Mathf.Infinity; // eða 0
        int minIndex = int.MaxValue;

        foreach (GameObject target in targets)
        {
            int index = target.GetComponent<Enemy>().index;
            float distance = target.GetComponent<Enemy>().distance;

            if (index < minIndex || (index == minIndex && distance > maxDistance))
            {
                minIndex = index;
                maxDistance = distance;
                MainTarget = target;
            }
        }

        return MainTarget;
    }
    private GameObject CheckStrongest()
    {
        int mostHealth = 0;

        foreach (GameObject target in targets)
        {
            int health = target.GetComponent<Enemy>().health;

            if (health > mostHealth)
            {
                mostHealth = health;
                MainTarget = target;
            }
        }

        return MainTarget;
    }
    private GameObject CheckWeakest()
    {
        int leastHealth = int.MaxValue;

        foreach (GameObject target in targets)
        {
            int health = target.GetComponent<Enemy>().health;

            if (health < leastHealth)
            {
                leastHealth = health;
                MainTarget = target;
            }
        }

        return MainTarget;
    }
    private GameObject CheckClosest()
    {
        float minDistanceFromTurret = Mathf.Infinity;

        foreach (GameObject target in targets)
        {
            float distance = Mathf.Sqrt(Mathf.Abs(transform.position.x - target.transform.position.x) + Mathf.Abs(transform.position.y - target.transform.position.y));

            if (distance < minDistanceFromTurret)
            {
                minDistanceFromTurret = distance;
                MainTarget = target;
            }
        }
        return MainTarget;
    } 
}
```

## TowerPlacement.cs
```csharp
using System;
using NUnit.Framework;
using UnityEngine;
using System.Collections.Generic;
using TMPro;

public class TowerPlacement : MonoBehaviour
{

    [SerializeField] private SpriteRenderer rangeSprite;
    [SerializeField] private CircleCollider2D rangeCollider;

    [SerializeField] private Color gray;
    [SerializeField] private Color red;

    [NonSerialized] public bool isPlacing = true;

    [NonSerialized] public List<GameObject> RestrictedAreas = new List<GameObject>();

    [SerializeField] private Tower tower;

    [SerializeField] private AudioClip placingSound;
    private AudioSource audiosource;

    void Awake()
    {
        rangeCollider.enabled = false;
        audiosource = GetComponent<AudioSource>();
    }
    void Update()
    {
        if (isPlacing)
        {
            Vector2 mousePosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);

            transform.position = mousePosition;
        }

        if (Input.GetMouseButtonDown(0) && RestrictedAreas.Count == 0 && isPlacing && tower.cost <= Player.main.Money)
        {
            rangeCollider.enabled = true;
            isPlacing = false;
            Player.main.Money -= tower.cost;

            if (MusicPlayer.main.sound)
            {
                audiosource.PlayOneShot(placingSound);
            }

            GetComponent<TowerPlacement>().enabled = false;
        }

        if (RestrictedAreas.Count > 0)
        {
            rangeSprite.color = red;
        }
        else { 
            rangeSprite.color = gray; 
        }
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if((collision.gameObject.tag == "Restricted" || collision.gameObject.tag == "Tower") && isPlacing == true)
        {
            RestrictedAreas.Add(collision.gameObject);
        }
    }
    private void OnTriggerExit2D(Collider2D collision)
    {
        if ((collision.gameObject.tag == "Restricted" || collision.gameObject.tag == "Tower") && isPlacing == true)
        {
            RestrictedAreas.Remove(collision.gameObject);
        }
    }
}

```

## TowerUpgrades.cs
```csharp
using System;
using UnityEngine;

public class TowerUpgrades : MonoBehaviour
{
    [System.Serializable] // basically við getum sjá Level class í inspector 
    public class Level
    {
        public float range = 10f;
        public int damage = 1;
        public float firerate = 1f;
        public int cost = 10;
    }

    public Level[] levels = new Level[3];
    [NonSerialized] public int currentLevel = 0;
    [NonSerialized] public string currentCost;

    private Tower tower;
    [SerializeField] private TowerRange towerRange;

    [SerializeField] private EffectsUpgrade effectsUpgrade;

    void Awake()
    {
        tower = GetComponent<Tower>();
        currentCost = "Cost: "+levels[0].cost.ToString();
    }

    public void Upgrade()
    {
        if (currentLevel< levels.Length && levels[currentLevel].cost <= Player.main.Money)
        {
            tower.range = levels[currentLevel].range;
            tower.damage = levels[currentLevel].damage;
            tower.firerate = levels[currentLevel].firerate;

            towerRange.updateRange();

            if (effectsUpgrade != null)
            {
                effectsUpgrade.Upgrade();
            }

            Player.main.Money -= levels[currentLevel].cost;
            currentLevel++;

            if (currentLevel >= levels.Length)
            {
                currentCost = "MAX";
            }
            else
            {
                currentCost = "Cost: " + levels[currentLevel].cost.ToString();
            }

        }
    }
}

```

--------------------------------------------------------------------------------------------------------------------------------------------------

# Tower Effects

## Effects.cs
```csharp
using System;
using UnityEngine;
using static UnityEngine.GraphicsBuffer;

public class Effects : MonoBehaviour
{
    [SerializeField] private GameObject explosion;
    public float explosionRadius = 1f;
    public int explosionDamage = 1;

    [SerializeField] private bool usingIce = false;
    public float slowDuration = 1f;
    public float slowIntensity = 0.1f;

    [SerializeField] private GameObject lightning;
    [NonSerialized] public int LightningDMG;
    public float lightningRadius = 1f;
    public int LightningChainCount = 2;

    [SerializeField] public bool usingMusic = false;
    public float firerateBoost = 0.1f;

    private void Awake()
    {
        if (lightning != null)
        {
            LightningDMG = GetComponent<Tower>().damage;
        }
    }
    public void ApplyEffectEnemy(GameObject enemy)
    {
        if (explosion != null && enemy != null)
        {
            GameObject explosionObject = Instantiate(explosion, enemy.transform.position, Quaternion.identity);
            Explosion explosionScript = explosionObject.GetComponent<Explosion>();
            explosionScript.explosionDamage = explosionDamage;
            explosionScript.explosionRadius = explosionRadius;
        }
        else if (usingIce && enemy != null) 
        {
            Enemy enemyScript = enemy.GetComponent<Enemy>();

            enemyScript.slowDuration = slowDuration;
            enemyScript.slowMultiplier = slowIntensity;
        }
        else if (lightning != null && enemy != null)
        {
            GameObject LightningObject = Instantiate(lightning, enemy.transform.position, Quaternion.identity);
            Lightning LightningScript = LightningObject.GetComponent<Lightning>();

            LightningScript.LightningChainCount = LightningChainCount-1;
            LightningScript.lightningRadius = lightningRadius;
            LightningScript.EnemyStruck.Add(enemy);
            LightningScript.LightningDMG = LightningDMG;
            LightningScript.EnemyStruckedByLightning = enemy;
        }
    }

    public void ApplyEffectTower(GameObject tower)
    {
        if (usingMusic && tower != null)
        {
            Tower boostedTower = tower.gameObject.GetComponent<Tower>();

            boostedTower.firerateMultiplier = firerateBoost;
        }
    }

    public void UnapplyEffectTower(GameObject tower)
    {
        if (usingMusic && tower != null)
        {
            Tower boostedTower = tower.gameObject.GetComponent<Tower>();

            boostedTower.firerateMultiplier = 0f;
        }
    }
}

```

## EffectsUpgrade.cs
```csharp
using System;
using UnityEngine;

public class EffectsUpgrade : MonoBehaviour
{

    [System.Serializable]
    class BombEffectLevels
    {
        public float explosionRadius = 1f;
        public int explosionDamage = 1;
    }

    [System.Serializable]
    class IceEffectLevels 
    {
        public float slowDuration = 1f;
        public float slowIntensity = 0.1f;
    }

    [System.Serializable]
    class LightningEffectLevels
    {
        public float lightningRadius = 1f;
        public int LightningChainCount = 2;
    }

    [System.Serializable]
    class MusicEffectLevels
    {
        public float firerateBoost = 0.1f;
    }

    [SerializeField] private BombEffectLevels[] BombTowerLevels = new BombEffectLevels[2];
    [SerializeField] private IceEffectLevels[] IceTowerLevels = new IceEffectLevels[2];
    [SerializeField] private LightningEffectLevels[] LightningTowerLevels = new LightningEffectLevels[2];
    [SerializeField] private MusicEffectLevels[] MusicTowerLevels = new MusicEffectLevels[3];

    [SerializeField] private bool usingExplosion = false;
    [SerializeField] private bool usingIce = false;
    [SerializeField] private bool usingLightning = false;
    [SerializeField] private bool usingMusic = false;

    [SerializeField] private TowerRange Range;

    [SerializeField] private Effects effects;

    [NonSerialized] public int currentLevel = 0;

    public void Upgrade()
    {
        if (currentLevel < BombTowerLevels.Length && usingExplosion)
        {
            effects.explosionDamage = BombTowerLevels[currentLevel].explosionDamage;
            effects.explosionRadius = BombTowerLevels[currentLevel].explosionRadius;

            currentLevel++;
        }
        else if (currentLevel < BombTowerLevels.Length && usingIce) 
        {
            effects.slowDuration = IceTowerLevels[currentLevel].slowDuration;
            effects.slowIntensity = IceTowerLevels[currentLevel].slowIntensity;

            currentLevel++;
        }
        else if (currentLevel < BombTowerLevels.Length && usingLightning)
        {
            effects.lightningRadius = LightningTowerLevels[currentLevel].lightningRadius;
            effects.LightningChainCount = LightningTowerLevels[currentLevel].LightningChainCount;

            currentLevel++;
        }
        else if (currentLevel < MusicTowerLevels.Length && usingMusic)
        {
            effects.firerateBoost = MusicTowerLevels[currentLevel].firerateBoost;

            for (int i = 0; i < Range.boostedTowers.Count; i++) // update-ar firerate boost fyrir alla towers í boostedTowers
            {
                effects.ApplyEffectTower(Range.boostedTowers[i]);
            }

            currentLevel++;
        }
    }


}

```

## Explosion.cs
```csharp
using System.Collections.Generic;
using UnityEngine;
using static UnityEngine.GraphicsBuffer;

public class Explosion : MonoBehaviour
{
    public int explosionDamage;
    public float explosionRadius;

    [SerializeField] private float explosionTime = 0.3f;
    private float timer = 0f;

    [SerializeField] private AudioClip explosionSound;
    private AudioSource audioSource;

    private List<GameObject> enemiesHitList = new List<GameObject>();

    private void Start()
    {
        transform.localScale = new Vector3(explosionRadius, explosionRadius, explosionRadius); // ??????? frá tutorial
        audioSource = GetComponent<AudioSource>();
        audioSource.PlayOneShot(explosionSound);
    }
    void Update()
    {
        if (timer>= explosionTime)
        {
            Destroy(gameObject);
        }
        else
        {
            timer += Time.deltaTime;
        }
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if(collision.gameObject.tag == "Enemy" && !enemiesHitList.Contains(collision.gameObject))
        {
            enemiesHitList.Add(collision.gameObject);
            collision.GetComponent<Enemy>().damageTaken(explosionDamage);
        }
    }
}

```

## Lightning.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using static UnityEngine.GraphicsBuffer;

public class Lightning : MonoBehaviour
{
    [NonSerialized] public float lightningRadius = 1f;
    [NonSerialized] public int LightningChainCount = 2;
    [NonSerialized] public int LightningDMG;

    [NonSerialized] public List<GameObject> EnemyStruck = new List<GameObject>();
    private List<GameObject> Enemies = new List<GameObject>();

    private LineRenderer linerenderer;

    [NonSerialized] public GameObject EnemyStruckedByLightning;
    private GameObject closestEnemy;

    private SpriteRenderer sprite;

    void Start()
    {
        transform.localScale = new Vector3(lightningRadius, lightningRadius, lightningRadius);
        sprite = GetComponent<SpriteRenderer>();
        sprite.enabled = false;
        linerenderer = GetComponent<LineRenderer>();
        linerenderer.positionCount = 2;

        StartCoroutine(findingEnemyAndSpawningLightning());
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.gameObject.tag == "Enemy")
        {
            Enemies.Add(collision.gameObject);
        }
    }

    private void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.gameObject.tag == "Enemy")
        {
            Enemies.Remove(collision.gameObject);
        }
    }
    IEnumerator findingEnemyAndSpawningLightning()
    {
        transform.position = EnemyStruckedByLightning.transform.position;

        yield return new WaitForSeconds(0.1f);

        float minDistance = Mathf.Infinity;

        //Debug.Log(Enemies.Count);
        for (int i = 0; i < Enemies.Count; i++)
        {
            float Distance = Mathf.Sqrt(Mathf.Abs(transform.position.x - Enemies[i].transform.position.x) + Mathf.Abs(transform.position.y - Enemies[i].transform.position.y));
            if (Distance < minDistance && !EnemyStruck.Contains(Enemies[i]))
            {
                minDistance = Distance;
                closestEnemy = Enemies[i];
            }
        }

        if (closestEnemy == null)
        {
            Destroy(gameObject);
        }
        else
        {
            closestEnemy.GetComponent<Enemy>().damageTaken(LightningDMG);

            linerenderer.SetPosition(0, gameObject.transform.position);
            linerenderer.SetPosition(1, closestEnemy.transform.position);

            LightningChainCount--;
            if (LightningChainCount > 0)
            {
                GameObject LightningObject = Instantiate(gameObject, closestEnemy.transform.position, Quaternion.identity);
                Lightning LightningScript = LightningObject.GetComponent<Lightning>();

                EnemyStruck.Add(closestEnemy);

                LightningScript.LightningChainCount = LightningChainCount;
                LightningScript.EnemyStruck = EnemyStruck;
                LightningScript.lightningRadius = lightningRadius;
                LightningScript.LightningDMG = LightningDMG;
                LightningScript.EnemyStruckedByLightning = closestEnemy;
            }

            yield return new WaitForSeconds(0.1f);
            Destroy(gameObject);
        }     
    }
}

```

--------------------------------------------------------------------------------------------------------------------------------------------------

# Enemy

## EnemyManager.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyManager : MonoBehaviour
{
    public static EnemyManager main;

    public Transform[] checkpoints;
    public Transform spawnpoint;

    [SerializeField] private GameObject wavePanel;

    [SerializeField] private int waveSet;
    [SerializeField] private int wave = 1; // þarf ekki serializefield en ég notar hann til að test-a

    private bool finishedSpawning = false;
    private bool waveFinished = false;

    void Awake()
    {
        main = this;
    }
    private void Start()
    {
        EnemyWaveSet.main.setWave(waveSet);

        StartCoroutine(Spawn(EnemyWaveSet.main.Waves[wave-1],3));
    }
    private void Update()
    {
        GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");

        if (wave>=EnemyWaveSet.main.Waves.Count && finishedSpawning && enemies.Length == 0)
        {
            Player.main.Finish();
            GetComponent<EnemyManager>().enabled = false; 
        }

        if (!waveFinished && finishedSpawning && enemies.Length == 0 && wave<= EnemyWaveSet.main.Waves.Count)
        {
            Player.main.Money += Mathf.RoundToInt(5 + (10 * (wave / 10)));
            waveFinished = true;
            wavePanel.SetActive(true);
        }
    }
    public void NextWave()
    {
        GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");

        wavePanel.SetActive(false);
        if (finishedSpawning == true && enemies.Length == 0 && EnemyWaveSet.main.Waves.Count > wave)
        {
            wave++;
            finishedSpawning = false;
            waveFinished = false;

            StartCoroutine(Spawn(EnemyWaveSet.main.Waves[wave - 1], 1));
        }
    }

    IEnumerator Spawn(EnemyWaveSet.Wave wave, float WaveInterval) //get nota await en ég er of latur :þ
    {
        yield return new WaitForSeconds(WaveInterval);

        for (int x = 0; x < wave.Enemies.Count; x++)
        {

            yield return new WaitForSeconds(wave.Enemies[x].SpawnDelay);
            for (int i = 0; i < wave.Enemies[x].EnemyCount; i++)
            {

                Instantiate(wave.Enemies[x].EnemyType, spawnpoint.position, Quaternion.identity);
                yield return new WaitForSeconds(wave.Enemies[x].SpawnInterval); // örugg þarf að breyta þetta
            }
        }
        finishedSpawning = true;
    }
}  
```

## EnemyWaveSet.cs
```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class EnemyWaveSet : MonoBehaviour
{
    public static EnemyWaveSet main;

    [SerializeField] private GameObject Basic_Enemy;
    [SerializeField] private GameObject Fast_Enemy;
    [SerializeField] private GameObject Strong_Enemy;
    [SerializeField] private GameObject Super_Basic_Enemy;
    [SerializeField] private GameObject Speedster_Enemy;
    [SerializeField] private GameObject Brute_Enemy;
    [SerializeField] private GameObject Commander_Enemy;
    [SerializeField] private GameObject Magician_Enemy;
    [SerializeField] private GameObject Summoner_Enemy;
    [SerializeField] private GameObject Gunner_Enemy;
    public class EnemySpawnData
    {
        public GameObject EnemyType { get; set; }
        public float SpawnDelay { get; set; }
        public int EnemyCount { get; set; }
        public float SpawnInterval { get; set; }

        public EnemySpawnData(GameObject enemyType, float spawnDelay, int enemycount, float spawnInterval)
        {
            EnemyType = enemyType;
            SpawnDelay = spawnDelay;
            EnemyCount = enemycount;
            SpawnInterval = spawnInterval;
        }
    }

    public class Wave
    {
        public int WaveNumber { get; set; }
        public List<EnemySpawnData> Enemies { get; set; }

        public Wave(int wavenumber)
        {
            WaveNumber = wavenumber;
            Enemies = new List<EnemySpawnData>();
        }
    }

    [NonSerialized] public List<Wave> Waves = new List<Wave>();

    // get örugglega nota dictionaries en það er bara 10 waves sooooo.

    private Wave wave1 = new Wave(1);
    private Wave wave2 = new Wave(2);
    private Wave wave3 = new Wave(3);
    private Wave wave4 = new Wave(4);
    private Wave wave5 = new Wave(5);
    private Wave wave6 = new Wave(6);
    private Wave wave7 = new Wave(7);
    private Wave wave8 = new Wave(8);
    private Wave wave9 = new Wave(9);
    private Wave wave10 = new Wave(10);

    private void Awake()
    {
        main = this;
    }

    public void setWave(int levelNumber)
    {
        if (levelNumber >= 0 && levelNumber <= 9)
        {
            switch (levelNumber)
            {
                case 0:
                    // test waveset

                    wave1.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 0.2f, 5, 0.3f));
                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 6, 0.8f));

                    Waves.Add(wave1);
                    Waves.Add(wave2);

                    break;

                case 1:
                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 0.8f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 0.5f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 0.5f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 4, 0.5f));

                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 6, 0.5f));
                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 4f, 3, 2f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 1, 0f));
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.4f, 4, 1f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 1, 0f));
                    wave6.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 0.6f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 0.8f));

                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 3, 1f));
                    wave7.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 4, 0.8f));

                    wave8.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 4, .5f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.5f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1.5f, 5, 0.7f));

                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 3, 1f)); // kannski breytta þetta
                    wave9.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 2f));

                    wave10.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 5, 1f));
                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 1f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.5f, 1, 0f));

                    break;
                case 2:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));
                    wave2.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 2, 1.5f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 0.9f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 0.7f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 1f));
                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 5f, 3, 0.3f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 1, 0f)); 
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.7f, 4, 1.5f));

                    wave6.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 1, 0f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 3f, 4, 0.6f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 2f, 4, 0.4f));

                    break;
                case 3:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 6, 1f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 0.8f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 1f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 2f, 2, 4f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 0.5f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 3f));
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.4f, 3, 1f));

                    wave6.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 2, 1f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 3f, 6, 0.8f));

                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 3, 1f));
                    wave7.Enemies.Add(new EnemySpawnData(Fast_Enemy, 5f, 5, 0.5f));

                    wave8.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 0.5f, 3, 2f));
                    wave8.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 5, 0.7f));

                    wave9.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 0f));
                    wave9.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 4, 1.5f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 1, 0f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 5f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.3f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.5f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 0.5f, 4, 1f));

                    break;
                case 4:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 1f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 8, 0.8f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 2f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 2f, 4, 0.5f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 2f));
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.6f, 7, 0.8f));
                    wave5.Enemies.Add(new EnemySpawnData(Fast_Enemy, 0.3f, 3, 0.5f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 5, 1.5f));
                    wave6.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 1f, 1, 0f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 5, 0.4f));

                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 4, 1f));
                    wave7.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 2f, 1, 0f));
                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 1f, 2, 2f));
                    wave7.Enemies.Add(new EnemySpawnData(Fast_Enemy, 3f, 6, 1f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Fast_Enemy, 2f, 20, 0.6f));

                    wave9.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 2, 2f));
                    wave9.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.3f, 3, 0.7f));
                    wave9.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 1f, 1, 1f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 2f, 1, 1.5f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 1.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.3f, 8, 0.3f));
                    wave9.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 0.7f, 3, 1f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.5f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 3.5f, 3, 1.5f));

                    break;
                case 5:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));
                    wave2.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 0.8f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 1f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 0.5f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 1f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 2.5f, 5, 1f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 1f));
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 1.8f, 7, 1f));
                    wave5.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1.3f, 5, 0.4f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 9, 0.8f));

                    wave7.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 0f));
                    wave7.Enemies.Add(new EnemySpawnData(Strong_Enemy, 2f, 5, 1.2f));
                    wave7.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 3f, 2, 1f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 2, 1.5f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 2f, 6, 0.2f));

                    wave9.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 4, 1f));
                    wave9.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.15f, 3, 1f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 2f, 2, 1.5f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 6, 1.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.3f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.3f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 5, 1f));
                    wave10.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 0.5f, 4, 0.4f));

                    break;
                case 6:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 2, 1f));
                    wave2.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 4, 1.2f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 6, 1f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 1f));
                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 2.5f, 6, 0.8f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 1, 0f));
                    wave5.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.4f, 2, 0.8f));
                    wave5.Enemies.Add(new EnemySpawnData(Fast_Enemy, 7f, 5, 1f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 4, 1.5f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 8f, 10, 0.5f));

                    wave7.Enemies.Add(new EnemySpawnData(Fast_Enemy, 0.5f, 6, 0.5f));
                    wave7.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 1f, 2, 0.8f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 1f, 6, 0.5f));

                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 5, 0.7f));
                    wave9.Enemies.Add(new EnemySpawnData(Commander_Enemy, 2f, 2, 1f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1.5f, 8, 0.5f));

                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.5f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.2f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 2, 0.3f));
                    wave10.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 1.5f, 8, 0.4f));

                    break;
                case 7:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 1.5f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 1.3f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 3, 1f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1.5f, 4, 0.9f));

                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 0.5f, 2, 0.4f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 1f));
                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 0.5f, 3, 0.6f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 1f));
                    wave5.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 4, 0.7f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 6, 1.5f));
                    wave6.Enemies.Add(new EnemySpawnData(Fast_Enemy, 2f, 9, 0.6f));

                    wave7.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 0.7f));
                    wave7.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 2, 0.8f));
                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 1f, 5, 0.4f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 2, 1f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 3, 0.8f));
                    wave8.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 2.5f, 5, 0.3f));

                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 9, 0.4f));
                    wave9.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 2, 0.5f));
                    wave9.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 2.5f, 4, 0.4f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 0.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 0.3f, 2, 0.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.2f, 3, 0.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 5, 0.3f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.3f, 3, 0.7f));
                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 0f));

                    break;
                case 8:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 1.5f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 6, 1.5f));

                    wave3.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 1.5f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 2f, 3, 0.4f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 1, 0f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 1f, 5, 0.4f));
                    wave4.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1.5f, 6, 0.5f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 5, 0.5f));
                    wave5.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 1, 0f));
                    wave5.Enemies.Add(new EnemySpawnData(Fast_Enemy, 3f, 4, 0.2f));

                    wave6.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 1f));
                    wave6.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 2f, 1, 0f));
                    wave6.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 1, 0f));

                    wave7.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 0.8f));
                    wave7.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 3, 0.4f));
                    wave7.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 1, 0f));
                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 1f, 1, 0f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 4, 1.5f));
                    wave8.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 1f, 3, 0.8f));

                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 5, 0.3f));
                    wave9.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 1, 0.5f));
                    wave9.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 1, 0f));
                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 5, 0.3f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.6f, 1, 0f));
                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 5, 0.3f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 0.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.5f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 3, 0.5f));
                    wave10.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 0.9f, 3, 1f));
                    wave10.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 3, 0.3f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.3f, 1, 0.7f));
                    wave10.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 1.5f, 3, 1f));

                    break;
                case 9:

                    // adding enemies to waves

                    // enemytype, spawndelay, amount, spawninterval

                    wave1.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 1.8f));

                    wave2.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 4, 1.5f));

                    wave3.Enemies.Add(new EnemySpawnData(Basic_Enemy, 0.5f, 5, 1.5f));
                    wave3.Enemies.Add(new EnemySpawnData(Fast_Enemy, 1f, 3, 0.9f));

                    wave4.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 0.8f));
                    wave4.Enemies.Add(new EnemySpawnData(Basic_Enemy, 3f, 5, 0.6f));

                    wave5.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 2, 1f));
                    wave5.Enemies.Add(new EnemySpawnData(Basic_Enemy, 1f, 4, 1f));
                    wave5.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 1, 0f));

                    wave6.Enemies.Add(new EnemySpawnData(Strong_Enemy, 0.5f, 3, 1.5f));
                    wave6.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 2f, 1, 1f));

                    wave7.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 2, 1.5f));
                    wave7.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.5f, 1, 0.8f));
                    wave7.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 1f, 2, 0.7f));
                    wave7.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 1f, 1, 0f));

                    wave8.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 1, 5f));
                    wave8.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.5f, 1, 0f));
                    wave8.Enemies.Add(new EnemySpawnData(Speedster_Enemy, 0.7f, 8, 0.4f));

                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 4, 0.3f));
                    wave9.Enemies.Add(new EnemySpawnData(Commander_Enemy, 1f, 1, 0.5f));
                    wave9.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 1f, 2, 0.5f));
                    wave9.Enemies.Add(new EnemySpawnData(Super_Basic_Enemy, 0.5f, 4, 0.3f));

                    wave10.Enemies.Add(new EnemySpawnData(Brute_Enemy, 0.5f, 9, 0.8f));
                    wave10.Enemies.Add(new EnemySpawnData(Summoner_Enemy, 0.8f, 1, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Gunner_Enemy, 0.5f, 3, 0f));
                    wave10.Enemies.Add(new EnemySpawnData(Magician_Enemy, 0.1f, 5, 0.3f));
                    wave10.Enemies.Add(new EnemySpawnData(Commander_Enemy, 0.3f, 1, 0.7f));

                    break;
            }

            // adding waves to waves list

            if(levelNumber>0 && levelNumber<=9) // því test waveset er er með Waves.add() í if statement (ln: 73)
            {
                Waves.Add(wave1);
                Waves.Add(wave2);
                Waves.Add(wave3);
                Waves.Add(wave4);
                Waves.Add(wave5);
                Waves.Add(wave6);
                Waves.Add(wave7);
                Waves.Add(wave8);
                Waves.Add(wave9);
                Waves.Add(wave10);
            }
        }
    }
}

```

## Enemy.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Enemy : MonoBehaviour
{

    [SerializeField] private float movementSpeed = 2f; // serializefield sýnir þetta í unity editor
    [SerializeField] private int value = 2;
    public int health = 4;

    private EnemyEffects effects;

    [SerializeField] private EnemyRange range;

    private float CDTimer = 0f;
    private float CD;

    [NonSerialized] public float slowMultiplier = 0f;
    [NonSerialized] public float slowDuration = 0f;

    [NonSerialized] public float boostMultiplier = 0f;

    [NonSerialized] public float dmgReduction = 0f;

    [SerializeField] private GameObject GunPart; // fyrir Gunner
    [SerializeField] private AudioClip shootingSound;

    [SerializeField] private GameObject slowIndicator;
    [NonSerialized] public SpriteRenderer slowRenderer;

    private Rigidbody2D RB;

    private Transform checkpoint;

    private AudioSource audioSource;

    // [SerializeField] AudioClip deathSound; // unused sound

    [NonSerialized] public  int index = 0; // nonserialized sýnir þetta ekki í unity editor
    [NonSerialized] public float distance = 0f;

    private void Awake()
    {
        RB = GetComponent<Rigidbody2D>();
        effects = GetComponent<EnemyEffects>();

        if (slowIndicator != null)
        {
            slowRenderer = slowIndicator.GetComponent<SpriteRenderer>();
            slowRenderer.enabled = false;
        }

        if(effects!= null)
        {
            if (effects.isShooter)
            {
                CD = effects.shooterCD;
            }
            else if (effects.isSummoner)
            {
                CD = effects.summonCD;
            }
        }

        if(GetComponent<AudioSource>() != null)
        {
            audioSource = GetComponent<AudioSource>();
        }
    }
    void Start()
    {
        checkpoint = EnemyManager.main.checkpoints[index];
    }
    private void Update()
    {
        distance = Vector2.Distance(transform.position, checkpoint.position);

        if (distance <= 0.1f) 
        {
            index++;
            if (index < EnemyManager.main.checkpoints.Length)
            {
                checkpoint = EnemyManager.main.checkpoints[index];
            }

            else if (index >= EnemyManager.main.checkpoints.Length)
            {
                Player.main.damage((health));
                Destroy(gameObject); 
            } 
        }

        if (slowDuration > 0f)
        {
            slowRenderer.enabled = true;
            slowDuration -= Time.deltaTime;

            if (slowDuration <= 0f)
            {
                slowDuration = 0f;
                slowMultiplier = 0f;
                slowRenderer.enabled = false;
            }
        }
        
        if (CDTimer < CD && effects != null)
        {
            if (effects.finishedSummoning) // default true
            {
                CDTimer += Time.deltaTime;
            }
        }

        else if (CDTimer >= CD && effects != null) // kannski þarf ekki (effects != null) því CDTimer mun aldrei hækka en oh well
        {
            // apply effect
            if (effects.isShooter)
            {
                GameObject target = range.MainTarget;
                if (target != null && GunPart != null)
                {
                    Vector2 dir = (target.transform.position - transform.position);
                    dir.Normalize();

                    float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;

                    GunPart.transform.rotation = Quaternion.Euler(0f, 0f, angle);

                    if(audioSource != null && shootingSound != null && MusicPlayer.main.sound)
                    {
                        audioSource.PlayOneShot(shootingSound);
                    }

                    effects.ApplyEffectOnTower(range.MainTarget);
                }
            }
            else if (effects.isSummoner && effects.finishedSummoning)
            {
                effects.ApplyEffect();
            }

            CDTimer = 0f;
        }
    }
    void FixedUpdate()
    {
        Vector2 dir = (checkpoint.position - transform.position);
        dir.Normalize();

        float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;
        transform.rotation = Quaternion.Euler(0f, 0f, angle); // fékk aðstoð frá yt

        RB.position += dir * movementSpeed * ((1 + boostMultiplier) - slowMultiplier) * Time.deltaTime;

        // RB.position += movementSpeed*dir // mjög hratt
    }

    public void damageTaken(int damage)
    {
        health -= Mathf.RoundToInt(damage*(1-dmgReduction));

        if (health <= 0)
        {
            if(effects != null)
            {
                if (effects.isCommander)
                {
                    List<GameObject> listi = range.boostedEnemies;

                    for (int i = 0; i < listi.Count; ++i)
                    {
                        Enemy BoostedEnemy = listi[i].gameObject.GetComponent<Enemy>();
                        BoostedEnemy.boostMultiplier = 0f;
                    }
                }
                else if (effects.isMagician)
                {
                    GameObject target = range.MainTarget;
                    if (target != null)
                    {
                        effects.unApplyEffectOnEnemy(range.MainTarget);
                    }
                }
            }

            Player.main.Money += value;
            Destroy(gameObject);
        }
    }
}

```

## EnemyEffects.cs
```csharp
using System;
using System.Collections;
using UnityEngine;

public class EnemyEffects : MonoBehaviour
{
    public bool isCommander = false;
    public bool isShooter = false;
    public bool isMagician = false;
    public bool isSummoner = false;

    [SerializeField] private float commanderBoost = 0.2f;

    [SerializeField] private float magicianDmgReduction = 0.5f;

    [SerializeField] private float shooterStunDuration = 2f;
    public float shooterCD = 5f; 

    [SerializeField] private int summonAmount = 3;
    [SerializeField] private float summonInterval = 0.5f;

    [SerializeField] private GameObject summonEnemy;

    [NonSerialized] public bool finishedSummoning = true;

    public float summonCD = 3f;

    public void ApplyEffect() 
    {
        if (isSummoner)
        {
            finishedSummoning = false;
            StartCoroutine(SummonEnemy());
        }
    }
    public void ApplyEffectOnTower(GameObject Tower)
    {
        Tower EffectedTower = Tower.GetComponent<Tower>();
        if (isShooter)
        {
            EffectedTower.stunDuration = shooterStunDuration;
            EffectedTower.stunRenderer.enabled = true;
        }
    }
    public void ApplyEffectOnEnemy(GameObject Enemy)
    {
        Enemy BoostedEnemy = Enemy.gameObject.GetComponent<Enemy>();
        if (isCommander)
        {
            BoostedEnemy.boostMultiplier = commanderBoost;
        }
        else if (isMagician)
        {
            BoostedEnemy.dmgReduction = magicianDmgReduction;
        }
    }
    public void unApplyEffectOnEnemy(GameObject Enemy)
    {
        Enemy BoostedEnemy = Enemy.gameObject.GetComponent<Enemy>();
        if (isCommander)
        {
            BoostedEnemy.boostMultiplier = 0f;
        }
        else if (isMagician)
        {
            BoostedEnemy.dmgReduction = 0f;
        }
    }
    IEnumerator SummonEnemy()
    {
        for (int i = 0;i<summonAmount;i++)
        {
            GameObject summonedEnemy = Instantiate(summonEnemy, gameObject.transform.position, Quaternion.identity);
            summonedEnemy.GetComponent<Enemy>().index = gameObject.GetComponent<Enemy>().index;
            yield return new WaitForSeconds(summonInterval);
        }
        finishedSummoning = true;
    }
}
```

## EnemyRange.cs
```csharp
using System;
using System.Collections.Generic;
using UnityEngine;
using static UnityEngine.GraphicsBuffer;

public class EnemyRange : MonoBehaviour
{
    [SerializeField] private Enemy Enemy;

    [SerializeField] private EnemyEffects effects;
    [SerializeField] private GameObject shield;
    private SpriteRenderer shieldRenderer;

    [NonSerialized] public List<GameObject> boostedEnemies = new List<GameObject>();
    private List<GameObject> TowerTargets = new List<GameObject>();

    [NonSerialized] public GameObject MainTarget;
    private GameObject previousTarget;

    [SerializeField] private float enemyRange = 1f;

    private AudioSource audioSource;
    
    [SerializeField] private AudioClip shieldingSound;

    private void Awake()
    {
        transform.localScale = new Vector3(enemyRange, enemyRange, enemyRange); // frá tutorial
        if(shield != null)
        {
            shieldRenderer = shield.GetComponent<SpriteRenderer>();
            shieldRenderer.enabled = false;
        }
        if (GetComponent<AudioSource>() != null)
        {
            audioSource = GetComponent<AudioSource>();
        }
    }

    private void Update()
    {
        if (effects != null)
        {
            if (effects.isMagician && boostedEnemies.Count > 0)
            {
                /* 
                // closest enemy
                MainTarget = checkClosestEnemy();
                if (previousTarget != MainTarget && previousTarget != null)
                {
                    effects.unApplyEffectOnEnemy(previousTarget);
                }
                if (MainTarget.GetComponent<Enemy>().dmgReduction == 0f)
                {
                    shieldRenderer.enabled = true;
                    effects.ApplyEffectOnEnemy(MainTarget);
                }

                shield.transform.position = MainTarget.transform.position;
                previousTarget = MainTarget;
                */

                // first closest enemy
                if (MainTarget == null && boostedEnemies.Count > 0)
                {
                    MainTarget = checkClosestEnemy();
                    if(MainTarget != null)
                    {
                        shield.transform.position = MainTarget.transform.position;
                        shieldRenderer.enabled = true;
                        if (audioSource != null && shieldingSound != null && MusicPlayer.main.sound)
                        {
                            audioSource.PlayOneShot(shieldingSound);
                        }
                        effects.ApplyEffectOnEnemy(MainTarget);
                    }
                }
                else if (!boostedEnemies.Contains(MainTarget))
                {
                    shieldRenderer.enabled = false;
                    effects.unApplyEffectOnEnemy(MainTarget);
                    MainTarget = null;
                }
                else
                {
                    shield.transform.position = MainTarget.transform.position;
                }
            }
            else if (effects.isMagician && boostedEnemies.Count == 0)
            {
                shieldRenderer.enabled = false;
            }

            if (effects.isShooter && TowerTargets.Count > 0)
            {
                MainTarget = checkClosestTower();
            }
        }
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (effects != null)
        {
            if (collision.gameObject.tag == "Tower" && effects.isShooter)
            {
                if (collision.gameObject.GetComponent<Tower>().isSupport == false)
                {
                    TowerTargets.Add(collision.gameObject);
                }
            }
            else if (collision.gameObject.tag == "Enemy" && (effects.isMagician || effects.isCommander))
            {
                boostedEnemies.Add(collision.gameObject);
                if (effects.isCommander)
                {
                    effects.ApplyEffectOnEnemy(collision.gameObject);
                }
            }
        }
    }
    private void OnTriggerExit2D(Collider2D collision)
    {
        if (effects != null)
        {
            if (collision.gameObject.tag == "Tower" && effects.isShooter)
            {
                if (collision.gameObject.GetComponent<Tower>().isSupport == false)
                {
                    TowerTargets.Remove(collision.gameObject);
                }
            }
            else if (collision.gameObject.tag == "Enemy" && (effects.isMagician || effects.isCommander))
            {
                boostedEnemies.Remove(collision.gameObject);
                if (effects.isCommander)
                {
                    effects.unApplyEffectOnEnemy(collision.gameObject);
                }
            }
        }

    }
    private GameObject checkClosestEnemy()
    {
        float minDistanceFromEnemy = Mathf.Infinity;

        foreach (GameObject target in boostedEnemies)
        {
            float distance = Mathf.Sqrt(Mathf.Abs(transform.position.x - target.transform.position.x) + Mathf.Abs(transform.position.y - target.transform.position.y));

            if (distance < minDistanceFromEnemy && target.name.Replace("(Clone)", "").Trim() != "Wizard enemy" && target.GetComponent<Enemy>().dmgReduction == 0f)
            {
                minDistanceFromEnemy = distance;
                MainTarget = target;
            }
        }
        return MainTarget;
    }
    private GameObject checkClosestTower()
    {
        float minDistanceFromTurret = Mathf.Infinity;

        foreach (GameObject target in TowerTargets)
        {
            float distance = Mathf.Sqrt(Mathf.Abs(transform.position.x - target.transform.position.x) + Mathf.Abs(transform.position.y - target.transform.position.y));

            if (distance < minDistanceFromTurret && effects.isShooter && target.GetComponent<Tower>().stunDuration <= 0)
            {
                minDistanceFromTurret = distance;
                MainTarget = target;
            }
        }
        return MainTarget;
    }
}
```








