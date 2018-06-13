# kevin
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using UnityEngine.UI;


public class CuttingImage : MonoBehaviour
{
    //保存路径
    private string path;
    //裁剪图片起始坐标
    private float x;
    private float y;
    //鼠标起始点
    private Vector3 start;
    //鼠标终点
    private Vector3 end;
    //点击开始裁剪
    //public static bool isStartCut;
    //是否结束裁剪
    private bool isEnd;
    //射线
    private Ray ray;
    private RaycastHit hit;
    //相机
    private Camera cam;
    //抠图原始位置
    // Use this for initialization
    void Start()
    {
        My_pinch pinch = FindObjectOfType<My_pinch>();
        cam = pinch.cam;
        path = Application.persistentDataPath + "/ImageCut/";
        Init();
    }

    //初始化创建保存裁剪图片文件夹
    public void Init()
    {
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
    }

    private GameObject[] points =new GameObject[4];
	private int w;
	private int  h;
    private float wh;
    private float he;
    private Vector3[] vecs=new Vector3[4];
	bool first=true;
	bool isOpen;
	bool  isop;

    void Update()
    {
        if (UIDesignControl.instance.isOn == 2)
        {
            ray = cam.ScreenPointToRay(Input.mousePosition);
            if (Physics.Raycast(ray, out hit))
            {
                if (Input.mousePosition.y <= Screen.height * 0.9f)
                {
                    if (Input.GetMouseButtonDown(0))
                    {
                        isOpen = true;
                        isop = false;
                        //开始截图
                        isEnd = true;
                        //鼠标起始点
                        start = hit.point;
                    }

                    if (isEnd && Input.GetMouseButton(0))
                    {
                        //鼠标结束点
                        end = hit.point;
                        //第一次拖动
                        if (first == true)
                        {
                            //截图框
                            wh = end.x - start.x;
                            he = end.y - start.y;
                            vecs[0] = new Vector3(start.x, start.y, start.z);
                            vecs[1] = new Vector3(wh + start.x, start.y, start.z);
                            vecs[2] = new Vector3(wh + start.x, he + start.y, start.z);
                            vecs[3] = new Vector3(start.x, he + start.y, start.z);
                            for (int i = 0; i < 4; i++)
                            {
                                if (points[i] == null)
                                {
                                    Object obj = Resources.Load("point");
                                    points[i] = GameObject.Instantiate(obj) as GameObject;

                                }
                                points[i].transform.position = vecs[i];
                                GameObject lines = points[i].GetComponentInChildren<line>().gameObject;
                                if (i - 1 >= 0)
                                {
                                    points[i].transform.LookAt(vecs[i - 1]);
                                    float del = Vector3.Distance(vecs[i], vecs[i - 1]);
                                    lines.transform.localScale = new Vector3(0.2f, 0.2f, del * 5);
                                    lines.transform.localPosition = new Vector3(0, 0, del * 5 / 2);
                                }
                                else
                                {
                                    points[i].transform.LookAt(vecs[3]);
                                    float del = Vector3.Distance(vecs[i], vecs[3]);
                                    lines.transform.localScale = new Vector3(0.2f, 0.2f, del * 5);
                                    lines.transform.localPosition = new Vector3(0, 0, del * 5 / 2);
                                }
                            }
                        }
                        else
                        {
                            //第二次拖动
                            int xxxx = 0;
                            float delta = 0;
                            for (int i = 0; i < 4; i++)
                            {
                                if (i == 0)
                                {
                                    float tempdelta = Vector3.Distance(end, vecs[i]);
                                    delta = tempdelta;
                                }
                                else
                                {
                                    float tempdelta = Vector3.Distance(end, vecs[i]);
                                    if (tempdelta < delta)
                                    {
                                        xxxx = i;
                                        delta = tempdelta;
                                    }
                                }
                            }
                            if (isOpen)
                            {
                                if (delta > 1.5f)
                                {
                                    //移动面
                                    xxxx = -1;
                                    ScaleAndRotate.isOpen = true;
                                    if (UIDesignControl.instance.ren.transform.childCount <= 0)
                                    {
                                        UIDesignControl.instance.ren.transform.parent.Find("cc")
                                            .SetParent(UIDesignControl.instance.ren.transform);
                                    }
                                    ScaleAndRotate sar = FindObjectOfType<ScaleAndRotate>();
                                    sar.GetPosInit();
                                }
                                else
                                {
                                    //移动点
                                    ScaleAndRotate.isOpen = false;
                                    if (UIDesignControl.instance.ren.transform.childCount > 0)
                                    {
                                        UIDesignControl.instance.ren.transform.Find("cc")
                                            .SetParent(UIDesignControl.instance.ren.transform.parent);
                                        UIDesignControl.instance.ren.transform.parent.Find("cc").SetSiblingIndex(2);
                                    }
                                }
                                isOpen = false;
                            }
                            if (Input.touchCount==1 && !ScaleAndRotate.isOpen)
                            {
                                if (xxxx == 0)
                                {
                                    start = vecs[2];
                                    wh = end.x - start.x;
                                    he = end.y - start.y;
                                    vecs[2] = new Vector3(start.x, start.y, start.z);
                                    vecs[3] = new Vector3(wh + start.x, start.y, start.z);
                                    vecs[0] = new Vector3(wh + start.x, he + start.y, start.z);
                                    vecs[1] = new Vector3(start.x, he + start.y, start.z);
                                }
                                else if (xxxx == 1)
                                {
                                    start = vecs[3];
                                    wh = end.x - start.x;
                                    he = end.y - start.y;
                                    vecs[3] = new Vector3(start.x, start.y, start.z);
                                    vecs[0] = new Vector3(wh + start.x, start.y, start.z);
                                    vecs[1] = new Vector3(wh + start.x, he + start.y, start.z);
                                    vecs[2] = new Vector3(start.x, he + start.y, start.z);
                                }
                                else if (xxxx == 2)
                                {
                                    start = vecs[0];
                                    wh = end.x - start.x;
                                    he = end.y - start.y;
                                    vecs[0] = new Vector3(start.x, start.y, start.z);
                                    vecs[1] = new Vector3(wh + start.x, start.y, start.z);
                                    vecs[2] = new Vector3(wh + start.x, he + start.y, start.z);
                                    vecs[3] = new Vector3(start.x, he + start.y, start.z);
                                }
                                else if (xxxx == 3)
                                {
                                    start = vecs[1];
                                    wh = end.x - start.x;
                                    he = end.y - start.y;
                                    vecs[1] = new Vector3(start.x, start.y, start.z);
                                    vecs[2] = new Vector3(wh + start.x, start.y, start.z);
                                    vecs[3] = new Vector3(wh + start.x, he + start.y, start.z);
                                    vecs[0] = new Vector3(start.x, he + start.y, start.z);
                                }
                                for (int i = 0; i < 4; i++)
                                {
                                    points[i].transform.position = vecs[i];
                                    GameObject lines = points[i].GetComponentInChildren<line>().gameObject;
                                    if (i - 1 >= 0)
                                    {
                                        points[i].transform.LookAt(vecs[i - 1]);
                                        float del = Vector3.Distance(vecs[i], vecs[i - 1]);
                                        lines.transform.localScale = new Vector3(0.2f, 0.2f, del * 5);
                                        lines.transform.localPosition = new Vector3(0, 0, del * 5 / 2);
                                    }
                                    else
                                    {
                                        points[i].transform.LookAt(vecs[3]);
                                        float del = Vector3.Distance(vecs[i], vecs[3]);
                                        lines.transform.localScale = new Vector3(0.2f, 0.2f, del * 5);
                                        lines.transform.localPosition = new Vector3(0, 0, del * 5 / 2);
                                    }
                                }
                            }
                        }
                    }
                    if (Input.GetMouseButtonUp(0))
                    {
                        first = false;
                    }
                }
            }
        }
    }


    #region CutImage

    //开始裁剪图片
    public void ImageCutting()
    {
        UIDesignControl.instance.ren.material.mainTextureScale=  new Vector2 (1, 1);
        first = true;
        UIDesignControl.instance.CloseButton(0);
        UIDesignControl.instance.operateImagelayoutGroupTrans.GetChild(2).GetComponent<Button>().enabled = true;
        UIDesignControl.instance.operateImagelayoutGroupTrans.GetChild(2).GetChild(0).GetComponent<Text>().color
            = UIDesignControl.instance.operateImagelayoutGroupTrans.GetChild(2).GetChild(1).GetComponent<Image>().color 
            = new Color(1, 1, 1, 1);

        if (UIDesignControl.instance.ren.transform.childCount <= 0)
        {
            UIDesignControl.instance.ren.transform.parent.Find("cc").SetParent(UIDesignControl.instance.ren.transform);
        }
        UIDesignControl.instance.isOn = 2;
        UIDesignControl.instance.OnShowNextButtonClick();
    }
    private Vector3 startPoint;
    private Vector3 endPoint;
    public void SureCut()
    {
        //截图
        if (UIDesignControl.instance.isOn==2)
        {
            //位置还原
            if (UIDesignControl.instance.ren.transform.childCount>0)
            {
                UIDesignControl.instance.ren.transform.Find("cc").SetParent(UIDesignControl.instance.ren.transform.parent);
                UIDesignControl.instance.ren.transform.parent.Find("cc").SetSiblingIndex(2);
            }
            for (int i = 0; i < 4; i++)
            {
                points[i].transform.SetParent(UIDesignControl.instance.ren.transform);
            }
            UIDesignControl.instance.ren.transform.localPosition = new Vector3(-0.1f, -1, 21.54f);
            UIDesignControl.instance.ren.transform.parent.Find("cc").localPosition = new Vector3(-0.1f, -1, 21.55f);
            UIDesignControl.instance.ren.transform.localScale = new Vector3(2, 1, 2);
            UIDesignControl.instance.ren.transform.parent.Find("cc").localScale = new Vector3(2.03f, 1, 2.03f);

            GameObject go = UIDesignControl.instance.ren.transform.parent.Find("cc").gameObject;
            for (int j = 1; j < LoadConstainsImageControl.instance.koutuPointGo.Length; j++)
            {
                LoadConstainsImageControl.instance.koutuPointGo[j] = go.transform.GetChild(j).position;
            }
            ScaleAndRotate.isOpen = false;
            UIDesignControl.instance.isRepeal = false;
            UIDesignControl.instance.OpenButton(0);
            //结束截图
            isEnd = false;
            //鼠标选框长和宽
            startPoint = cam.WorldToScreenPoint(points[0].transform.position);
            endPoint = cam.WorldToScreenPoint(points[2].transform.position);
            w = (int)Mathf.Abs(startPoint.x - endPoint.x);
			h = (int)Mathf.Abs(startPoint.y - endPoint.y);
            //删除点
            for (int i = 0; i < 4; i++)
            {
                Destroy(points[i]);
            }
            //裁剪图片
            StartCoroutine(GetCapture());
            Resources.UnloadUnusedAssets();
        }
    }

    //彩色截图
	private Texture2D main;
    IEnumerator GetCapture()
    {
        //等待所有的摄像机跟GUI渲染完成
        yield return new WaitForEndOfFrame();
        //新建画布
        Texture2D tex = new Texture2D(w, h, TextureFormat.ARGB32, false);

        //----------------------------------------------------------------------------计算区域----------------------------------------------------
        //取较小的x,y作为起始点
		float vx = startPoint.x > endPoint.x ? endPoint.x : startPoint.x;
		float vy = startPoint.y > endPoint.y ? endPoint.y : startPoint.y;
        //画布上截图
        tex.ReadPixels(new Rect(vx, vy, w, h), 0, 0, true);
        tex.Apply();
		main = tex;
		if (black == null) 
        {
			UIDesignControl.instance.ren.material.color = Color.black;
		} 
        else 
		{
			UIDesignControl.instance.ren.material.SetTexture("_MainTex", black );
        }
        StartCoroutine(GetCapture1());
        
    }

    //黑色范围图
	Texture2D  black;
	IEnumerator GetCapture1()
	{
		yield return new WaitForEndOfFrame();
        Texture2D tex = new Texture2D(w, h, TextureFormat.ARGB32, false);
		float vx = startPoint.x > endPoint.x ? endPoint.x : startPoint.x;
		float vy = startPoint.y > endPoint.y ? endPoint.y : startPoint.y;
        tex.ReadPixels(new Rect(vx, vy, w, h), 0, 0, true);
        tex.Apply();
        black = tex;
        if (w <= h)
		{
            UIDesignControl.instance.ren.transform.localScale = new Vector3(((float)w / (float)h) * UIDesignControl.instance.ktV3.x, UIDesignControl.instance.ktV3.y, UIDesignControl.instance.ktV3.z);
		}
		else
		{
			UIDesignControl.instance.ren.transform.localScale = new Vector3(UIDesignControl.instance.ktV3.x, UIDesignControl.instance.ktV3.y, ((float)h /(float ) w) * UIDesignControl.instance.ktV3.z);
		}
        UIDesignControl.instance.ren.material.SetTexture("_MainTex", main);
        UIDesignControl.instance.ren.material.SetTexture("_Mask", LoadConstainsImageControl.instance.blackTex);
        UIDesignControl.instance.ren.material.color = Color.white;
        StartCoroutine(GetCapture2());
	}

    //完整截图
    IEnumerator GetCapture2()
	{
		//等待所有的摄像机跟GUI渲染完成
		yield return new WaitForEndOfFrame();
		Vector3 wh1 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[0]);
		Vector3 wh2 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[1]);
		Vector3 wh3 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[2]);
		w = (int)(wh2.x - wh1.x);
		h = (int)(wh3.y - wh2.y);
		//新建画布
		Texture2D tex = new Texture2D(w, h, TextureFormat.ARGB32, false);
		//----------------------------------------------------------------------------计算区域----------------------------------------------------
		//画布上截图
		tex.ReadPixels(new Rect(wh1.x, wh1.y, w, h), 0, 0, true);
		tex.Apply();
	    main = tex;
		LoadConstainsImageControl.instance.texList.Add(tex);
		UIDesignControl.instance.ren.material.SetTexture("_MainTex", black);
		StartCoroutine (GetCapture3());
	}

    //黑白去色轮廓图
    [HideInInspector]
    public Texture2D blackTex;
    IEnumerator GetCapture3()
    {
        //等待所有的摄像机跟GUI渲染完成
        yield return new WaitForEndOfFrame();
        Vector3 wh1 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[0]);
        Vector3 wh2 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[1]);
        Vector3 wh3 = LoadConstainsImageControl.instance.cam.WorldToScreenPoint(LoadConstainsImageControl.instance.koutuPointGo[2]);
        int xx = (int)(wh2.x - wh1.x);
        int yy = (int)(wh3.y - wh2.y);
        // 新建画布
        blackTex = new Texture2D(xx, yy, TextureFormat.ARGB32, false);
        //----------------------------------------------------------------------------计算区域----------------------------------------------------
        //画布上截图
        blackTex.ReadPixels(new Rect(wh1.x, wh1.y, xx, yy), 0, 0, true);
        blackTex.Apply();

        LoadConstainsImageControl.instance.texBlackList.Add(blackTex);
        UIDesignControl.instance.ren.material.SetTexture("_Mask", blackTex);
        UIDesignControl.instance.ren.material.SetTexture("_MainTex", main);
      
        UIDesignControl.instance.ren.transform.localScale = new Vector3(UIDesignControl.instance.ktV3.x, UIDesignControl.instance.ktV3.y, UIDesignControl.instance.ktV3.z);
        Destroy();
    }

    private void Destroy()
    {
        for (int i = 0; i < RegionChoose.screenPos.Length; i++)
        {
            RegionChoose.screenPos[i] = Vector4.zero;
        }
    }

    #endregion

}
