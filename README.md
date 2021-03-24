# GameWorld
New game world when testing through the world. 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraTrack : MonoBehaviour
{
    Vector3 defalut_angle;

    public Vector3 cameraVector3 = new Vector3(0, 0, 0);
    public Transform cameraTransform;

    [Range(1, 10)] public float cameraSpeed = 5;

    public static float screenshake = 0;

    // Start is called before the first frame update
    void Start()
    {
        defalut_angle = cameraTransform.eulerAngles;
    }

    // Update is called once per frame
    void LateUpdate()
    {
        float time = Time.deltaTime;

        Vector3 goal_Position = transform.position + cameraVector3;

        cameraTransform.position = Vector3.Lerp(cameraTransform.position, goal_Position, time * cameraSpeed);
        cameraTransform.eulerAngles = defalut_angle;
        cameraTransform.Rotate(Random.Range(-screenshake, screenshake), Random.Range(-screenshake, screenshake), 0);

        screenshake = Mathf.Lerp(screenshake, 0, time * 10);
    }
}

// ---------------------------------------------------------------------------------------------------------------------------------------

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Controller : MonoBehaviour
{

    float speed = 0.5f;
    float rotationSpeed = 80;
    float rotation = 0f;
    float gravity = 8;

    Vector3 movingDirection = Vector3.zero;

    CharacterController controller;
    Animator animator;

    // Start is called before the first frame update
    void Start()
    {
        controller = GetComponent<CharacterController> ();
        animator = GetComponent<Animator> ();
    }

    // Update is called once per frame
    void Update()
    {
        Movement();
        GetInput();
    }

    void Movement()
    {
        // Start moving [Walking Animation]
        if (Input.GetKeyDown(KeyCode.W) || Input.GetKeyDown(KeyCode.UpArrow))
        {
            if (animator.GetBool("attacking") == true)
            {
                return;
            }
            else if (animator.GetBool("attacking") == false)
            {
                animator.SetBool("walking", true);
                animator.SetInteger("condition", 1);
                movingDirection = new Vector3(0, 0, 4);
                movingDirection *= speed;
                movingDirection = transform.TransformDirection(movingDirection);
            }
        }

        // Stop moving [Idel Animation]
        if (Input.GetKeyUp(KeyCode.W) || Input.GetKeyUp(KeyCode.UpArrow))
        {
            animator.SetBool("walking", false);
            animator.SetInteger("condition", 0);
            movingDirection = new Vector3(0, 0, 0);
        }

        rotation += Input.GetAxis("Horizontal") * rotationSpeed * Time.deltaTime;
        transform.eulerAngles = new Vector3(0, rotation, 0);

        movingDirection.y -= gravity * Time.deltaTime;
        controller.Move(movingDirection * Time.deltaTime);
    }

    void GetInput()
    {
        if (Input.GetMouseButtonDown(0))
        {
            if (animator.GetBool("walking") == true)
            {
                animator.SetBool("walking", false);
                animator.SetInteger("condition", 0);
            }

            if (animator.GetBool("walking") == false)
            {
                Attacking();
            }
        }
    }

    void Attacking()
    {
        StartCoroutine(AttackRoutine());
    }

    IEnumerator AttackRoutine()
    {
        animator.SetBool("attacking", true);
        animator.SetInteger("condition", 2);
        yield return new WaitForSeconds(1);
        animator.SetInteger("condition", 0);
        animator.SetBool("attacking", false);
    }
}

// ---------------------------------------------------------------------------------------------------------------------------------------

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DialogActivator : MonoBehaviour
{
    public string[] lines;

    private bool canActivate;

    public bool isPerson = true;

    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if (canActivate && Input.GetButtonDown("1")
           && !DialogManager.instance.dialogBox.activeInHierarchy)
        {
            DialogManager.instance.ShowDialog(lines, isPerson);
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.tag == "Player")
        {
            canActivate = true;
        }
    }

    public void OnTriggerExit(Collider other)
    {
        if (other.tag == "Player")
        {
            canActivate = false;
        }
    }
}

// ---------------------------------------------------------------------------------------------------------------------------------------

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class DialogManager : MonoBehaviour
{
    public Text dialogText;
    public Text nameText;
    public GameObject dialogBox;
    public GameObject nameBox;

    public string[] dialogLines;

    public int currentLines;

    private bool justStarted;

    public static DialogManager instance;

    // Start is called before the first frame update
    void Start()
    {
        instance = this;
    }

    // Update is called once per frame
    void Update()
    {
        if (dialogBox.activeInHierarchy)
        {
            if (!justStarted)
            {
                currentLines++;

                if (currentLines >= dialogLines.Length)
                {
                    dialogBox.SetActive(false);

                    GameManger.instance.dialogActive = false;
                }
                else
                {
                    CheckIfName();
                    dialogText.text = dialogLines[currentLines];
                }
            }
            else
            {
                justStarted = false;
            }
        }
    }

    public void ShowDialog(string[] newLines, bool isPerson)
    {
        dialogLines = newLines;
        currentLines = 0;
        CheckIfName();
        dialogText.text = dialogLines[currentLines];
        justStarted = true;

        nameBox.SetActive(isPerson);

        GameManger.instance.dialogActive = true;
    }

    public void CheckIfName()
    {
        if (dialogLines[currentLines].StartsWith("n-"))
        {
            nameText.text = dialogLines[currentLines].Replace("n-", "");
            currentLines++;
        }
    }
}

// ---------------------------------------------------------------------------------------------------------------------------------------

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class GameManger : MonoBehaviour
{
    public static GameManger instance;

    public bool dialogActive;

    // Start is called before the first frame update
    void Start()
    {
        instance = this;
        DontDestroyOnLoad(gameObject);
    }

    // Update is called once per frame
    void Update()
    {
        // When the player talks with the NPC, then the moving or walking stops. 
        if (dialogActive)
        {
            UpdatedController.instance.canMove = false; 
        }
        // if the case wasn't true, then the player will move.
        else
        {
            UpdatedController.instance.canMove = true;
        }
    }
}

// ---------------------------------------------------------------------------------------------------------------------------------------

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UpdatedController : MonoBehaviour
{
    Animator animPlayer;
    Rigidbody rbPlayer;

    public static UpdatedController instance;
    public bool canMove = true;

    // Start is called before the first frame update
    void Start()
    {
        if (instance == null)
        {
            instance = this;
        }
        else
        {
            if (instance != this)
            {
                Destroy(gameObject);
            }
        }
        DontDestroyOnLoad(gameObject);

        animPlayer = GetComponent<Animator>();
        rbPlayer = GetComponent<Rigidbody>();
    }

    // Update is called once per frame
    void Update()
    {
        // Walk: 
        if (canMove)
        {
            if (Input.GetKeyDown(KeyCode.W) || Input.GetKeyDown(KeyCode.S) ||
                Input.GetKeyDown(KeyCode.A) || Input.GetKeyDown(KeyCode.D))
            {
                animPlayer.SetBool("walking", true);
                animPlayer.SetBool("attacking", false);
                animPlayer.SetInteger("condiction", 1);
            }

            // Idle: 
            if (Input.GetKeyUp(KeyCode.W) || Input.GetKeyUp(KeyCode.S) ||
                Input.GetKeyUp(KeyCode.A) || Input.GetKeyUp(KeyCode.D))
            {
                animPlayer.SetBool("walking", false);
                animPlayer.SetBool("attacking", false);
                animPlayer.SetInteger("condiction", 0);
            }

            // Attack:
            if (Input.GetMouseButtonDown(0))
            {
                animPlayer.SetBool("walking", false);
                animPlayer.SetBool("attacking", true);
                animPlayer.SetInteger("condiction", 2);
            }
        }
    }

    private void FixedUpdate()
    {

        // Forward
        if (Input.GetKey(KeyCode.W))
        {
            rbPlayer.AddForce(new Vector3(0, 0, .5f), ForceMode.VelocityChange);
            rbPlayer.rotation = Quaternion.LookRotation(Vector3.forward);
        }

        // Backward
        if (Input.GetKey(KeyCode.S))
        {
            rbPlayer.AddForce(new Vector3(0, 0, -.5f), ForceMode.VelocityChange);
            rbPlayer.rotation = Quaternion.LookRotation(Vector3.back);
        }

        // Left-ward
        if (Input.GetKey(KeyCode.A))
        {
            rbPlayer.AddForce(new Vector3(-.5f, 0, 0), ForceMode.VelocityChange);
            rbPlayer.rotation = Quaternion.LookRotation(Vector3.left);
        }

        // Right-ward
        if (Input.GetKey(KeyCode.D))
        {
            rbPlayer.AddForce(new Vector3(.5f, 0, 0), ForceMode.VelocityChange);
            rbPlayer.rotation = Quaternion.LookRotation(Vector3.right);
        }
    }
}
