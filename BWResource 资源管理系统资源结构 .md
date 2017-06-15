# **BWResource 资源管理系统资源结构** 

## **资源结构**：

<Chunk>:` pChunkPtr->openSections("model", lModelPtrs)`

​	循环`LModelPtrs`：得到`LModelPtr`：

  - `resrouce = lModelPtr->ReadString("resource")`
  - Transform

​	<Model>：`GetVisual()`

​		<Visual>：`pVisualPtr->openSections("rednerSet", lRenderSets)`

​			循环`lRenderSets`：得到`lRenderSet`

​			<RenderSet>: `lRenderSet->openSections("geometry", lGeometryPtrs)`

​					循环`lGeometryPtrs`：得到`lGeometryPtr`

​					<Geometry>：

>     				1. `BinaryPtr lIndicesPtr = GetIndices(pGeometryPtr, lPrimitiveName)`
> 				2. Load Vertices:
>        				1. `BinaryPtr lVerticesPtr = GetVertexs(......)`
>        				2. std::vector<PrimGroup>lPrimGroups;
>        				3. std::vector<uint32>lIndexBuffer;
>        				4. `int nIndices = GetPrimGroups(lIndicesPtr, lPrimGroups,lIndexBuffer)`从这里能得到index buffer的值





## **Header结构**

1. Binary Indice File：

   ```c++
   IndexHeader{
     char IndexFormat[64];
     int nIndices;
     int nTriangleGroups;
   }
   之后即为
     Indices Data
   ```

   ​

2. Primitive Group:

   ```c++
   PrimitiveGourp{
     int startindex;
     int nPrimitives;
     int startvertex;
     int nvertices;
   }
   ```

   ​

3. Vertices:

   ```c++
   VertexHeader{
     char[64] vertexFormat;
     int nVertices;
   }
   之后即为
     Vertices Data
   ```

   ​