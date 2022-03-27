---
layout: post
title: Unity 快速合并Mesh
categories: [cate1, cate2]
description: 
keywords: 开发, 
---

一个小工具，快速的合并多个Mesh。

### 演示

![CombineMesh](/images/Unity/Editor/CombineMesh.gif)

### 实现

```
using UnityEngine;
using UnityEditor;
using System.Linq;
using System.Collections.Generic;
using System.Text;
public static class MeshCombineTool
{
    [MenuItem("工具/合并所选Mesh", true, 20)]
    public static bool VerifyMergeSelectedMeshes()
    {
        return Selection.transforms.SelectMany(x => x.GetComponentsInChildren<MeshFilter>())
                   .Where(x => x.sharedMesh != null).ToList().Count > 1;
    }

    [MenuItem("工具/合并所选Mesh")]
    public static void MergeSelectedMeshes()
    {
        List<MeshFilter> meshFilters = new List<MeshFilter>();
        List<Material> materials = new List<Material>();
        List<string> names = new List<string>();
        foreach (Transform t in Selection.transforms)
        {
            foreach (MeshFilter mf in t.GetComponentsInChildren<MeshFilter>())
            {
                if (mf == null || mf.sharedMesh == null) continue;

                meshFilters.Add(mf);

                MeshRenderer mr = mf.transform.GetComponent<MeshRenderer>();

                Debug.Log(mf.sharedMesh.name + ":" + mf.sharedMesh.subMeshCount);
                for (int i = 0; i < mf.sharedMesh.subMeshCount; i++)
                {
                    Material[] sharedMaterials = mr == null ? null : mr.sharedMaterials;

                    if (sharedMaterials != null && i < sharedMaterials.Length)
                        materials.Add(sharedMaterials[i]);
                    else
                        materials.Add(null);
                }
                if (!names.Contains(t.name))
                {
                    names.Add(t.name);
                }
            }
        }

        Mesh combined;
        CombineMesh(meshFilters, out combined);
        string combineName = CombineMeshName(names);

        string savePath =
            EditorUtility.SaveFilePanelInProject("保存合并后的Mesh文件", combineName, "asset", "保存Mesh");
        if (string.IsNullOrEmpty(savePath) || savePath.Equals("DO_NOT_SAVE"))
        {
            Debug.Log("取消保存");
            return;
        }
        AssetDatabase.CreateAsset(combined, savePath);
        Vector3 offset = CenterPivot(combined);
        GameObject go = new GameObject(combineName);
        Undo.RegisterCreatedObjectUndo(go, combineName);
        go.transform.position = offset;
        go.AddComponent<MeshFilter>().sharedMesh = combined;
        MeshRenderer ren = go.AddComponent<MeshRenderer>();
        ren.sharedMaterials = materials.ToArray();

        Selection.activeTransform = go.transform;
        EditorGUIUtility.PingObject(go);
    }

    private static T[] Fill<T>(int count)
    {
        T[] val = new T[count];
        for (int i = 0; i < count; i++)
            val[i] = default(T);
        return val;
    }

    private static string CombineMeshName(List<string> names)
    {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("Mesh[");
        for (int i = 0; i < names.Count; i++)
        {
            if (i != 0)
                stringBuilder.Append("-");
            stringBuilder.Append(names[i]);
        }
        stringBuilder.Append("]");
        return stringBuilder.ToString();
    }

    /// <summary>
    /// 合并mesh
    /// </summary>
    /// <param name="mesheFilters"></param>
    /// <param name="result"></param>
    /// <returns></returns>
    public static bool CombineMesh(IEnumerable<MeshFilter> mesheFilters, out Mesh result)
    {
        List<Vector3[]> vertices = new List<Vector3[]>();
        List<BoneWeight[]> boneWeights = new List<BoneWeight[]>();
        List<Color[]> colors = new List<Color[]>();
        List<Color32[]> colors32 = new List<Color32[]>();
        List<Vector3[]> normals = new List<Vector3[]>();
        List<Vector4[]> tangents = new List<Vector4[]>();
        List<Vector2[]> uv = new List<Vector2[]>();
        List<Vector2[]> uv2 = new List<Vector2[]>();
#if UNITY_5
        List<Vector2[]> uv3 = new List<Vector2[]>();
        List<Vector2[]> uv4 = new List<Vector2[]>();
#endif

        List<int[]> indices = new List<int[]>();
        List<MeshTopology> topology = new List<MeshTopology>();

        int vertexOffset = 0;
        foreach (MeshFilter filterItem in mesheFilters)
        {
            Mesh m = filterItem.sharedMesh;
            int vc = m.vertexCount;
            Vector3[] vrt = m.vertices;
            Vector3[] nrm = m.normals;

            // localposition 2 worldposition
            for (int i = 0; i < vc; i++)
            {
                vrt[i] = filterItem.transform.TransformPoint(vrt[i]); // 顶点
                nrm[i] = filterItem.transform.TransformDirection(nrm[i]); // 法线
            }

            vertices.Add(vrt);
            normals.Add(nrm);

            boneWeights.Add(
                m.boneWeights == null || m.boneWeights.Length < 1 ? Fill<BoneWeight>(vc) : m.boneWeights);
            colors.Add(m.colors == null || m.colors.Length < 1 ? Fill<Color>(vc) : m.colors);
            colors32.Add(m.colors32 == null || m.colors32.Length < 1 ? Fill<Color32>(vc) : m.colors32);
            tangents.Add(m.tangents == null || m.tangents.Length < 1 ? Fill<Vector4>(vc) : m.tangents);
            uv.Add(m.uv == null || m.uv.Length < 1 ? Fill<Vector2>(vc) : m.uv);
            uv2.Add(m.uv2 == null || m.uv2.Length < 1 ? Fill<Vector2>(vc) : m.uv2);
#if UNITY_5
            uv3.Add(uv3 == null || m.uv3.Length < 1 ? Fill<Vector2>(vc) : m.uv3);
            uv4.Add(uv3 == null || m.uv4.Length < 1 ? Fill<Vector2>(vc) : m.uv4);
#endif

            for (int i = 0; i < m.subMeshCount; i++)
            {
                int[] ind = m.GetIndices(i);

                for (int n = 0; n < ind.Length; n++)
                    ind[n] += vertexOffset;

                indices.Add(ind);

                topology.Add(m.GetTopology(i));
            }
            vertexOffset += vc;
        }

        if (vertexOffset > 64999)
        {
            result = null;
            return false;
        }

        result = new Mesh();

        result.vertices = vertices.SelectMany(x => x).ToArray();
        result.normals = normals.SelectMany(x => x).ToArray();
        // 顶点的骨骼权重
        result.boneWeights = boneWeights.SelectMany(x => x).ToArray();
        // 顶点颜色
        result.colors = colors.SelectMany(x => x).ToArray();
        result.colors32 = colors32.SelectMany(x => x).ToArray();
        result.tangents = tangents.SelectMany(x => x).ToArray();
        result.uv = uv.SelectMany(x => x).ToArray();
        result.uv2 = uv2.SelectMany(x => x).ToArray();
#if UNITY_5
        result.uv3 = uv3.SelectMany(x => x).ToArray();
        result.uv4 = uv4.SelectMany(x => x).ToArray();
#endif
        result.subMeshCount = indices.Count;

        for (int i = 0; i < indices.Count; i++)
        {
            // 子网格索引缓冲区
            result.SetIndices(indices[i], topology[i], i);
        }
        return true;
    }


    public static Vector3 CenterPivot(Mesh mesh)
    {
        Vector3[] v = mesh.vertices;
        Vector3 cen = mesh.bounds.center;

        for (int i = 0; i < v.Length; i++)
            v[i] -= cen;

        mesh.vertices = v;
        mesh.RecalculateBounds();
        return cen;
    }
}
```

