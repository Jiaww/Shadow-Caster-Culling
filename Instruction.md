# Shadow Caster Culling使用文档

## 简介：
   在*实时渲染*(Real-Time rendering)中，*阴影映射*(Shadow mapping)过程是限制渲染速度的一个非常重要的环节，因为在每一帧渲染之前，我们还需要对整个场景进行一次*深度缓冲*(depth buffer)的渲染(shadow pass)，用来检测物体是否在阴影中。
   本工具的作用是，针对该过程，对场景中的物体进行*离线*（off-line)的预处理，将那些在渲染中投射的阴影没有意义的物体标记出来，这样，在渲染过程中，就可以通过读取标记，在shadow
 pass过程中忽略这些物体，进而对渲染过程进行加速。

## 原理：
  之前提到有些物体在阴影映射中，它所投射的阴影是*“无意义”*的，这种*“无意义”*主要分以下两种情况：
1. 该物体完全被其他物体所遮挡：
在这种情况下，根据shadow mapping的原理，任何处于该物体阴影之下的物体同时也会完全处于其他物体的阴影之下，因此在shadow pass这种物体可以完全忽略。
2. 该物体所投射的阴影中并不存在其他物体：
在这种情况下，在渲染中，并不会有任何物体的颜色计算需要该物体所投射的阴影，因此该阴影也是没有意义的。（这种情况最多发生在地形(Terrain)上）

## **使用说明**：
- **环境设置**：
本工具使用Visual Studio 2012编译，所需要的库文件以及头文件均在工具根目录的include文件夹以及lib文件夹中，同时所需要的.dll文件均在工具根目录的debug文件夹中。
- **配置文档**：
程序的可配置文档是在根目录下的GlobalSetting.xml，用户可以通过修改配置文档来配置工具的输入与参数，其中可修改的输入与参数如下：
  1. 场景管理系统总目录(Resource_Path)：即所使用的场景管理系统BWResource的总目录，全部chunks，models等信息所保存的目录。
  2. 区块资源目录(Universe_Path)：即区块(chunks)等信息所在目录。
  3. 窗口以及高度映射图的大小(Window_HMap_size)：即窗口以及高度图的长和宽。
  4. 邻近区块数量(Num_Chunks)：在检测某一区块中的物体时，需要将周围的区块全部考虑在内，因为当前区块的物体可能会遮挡或被周围的区块的物体所遮挡。在这里可以设置需要考虑在内以当前区块为中心的邻近区块的行列数(left, right, top, bottom)。
  5. 光源集合(Light_Set)：因为光源可能存在多个或是一天的光源的位置及角度也会变化，因此本工具支持多光源输入，工具会逐一对不同光源进行检测，并将不同光源的结果写入.chunks文件的shadow字段中。
  6. 光源(Light)：对于多光源输入，直接可以通过在配置文档中按照下述样例格式添加<Light></Light>即可。光源的可配置的属性如下：
    a. 位置(Position)
    b. 方向(Direction)：通过设置光源的水平夹角(horizontal angle)与垂直夹角(vertical angle)。(关于方向的设置可参考下图)
    c. 投影类型(ProjectionType)：目前只支持Directional光(如有需求，很容易修改)
    d. 投影矩阵的参数(Projection_Parameters): 宽度(Width)，高度(Height)，近剪裁平面距离(Near)，远剪裁平面距离(Far)。
  7. 蒙版偏差(Stencil_Bias)：该属性决定了在蒙版写入时，需要剪裁掉地形高度以上多少偏差的物体。(为了避免地形高度信息保存在的texture精度不足的误差)
  8. 渲染延迟(Render_Delay)：该属性可以设置检测不同chunk或者不同光源下的渲染的间隔时间，因为在检测结束后会渲染出结果在窗口中，因此可以通过设置较大的间隔时间来观察检测结果(检测结果会用颜色标记)，或是设置为0达到最快的检测速度。(单位为毫秒ms)

  配置文档的基本格式如下：
  ```xml
   <?xml version="1.0" encoding="gb2312"?>    
  <Node>    
    <Header>    
      <EngName>Global Parameters Setting</EngName>    
    </Header>    
    <Parameters>    
      <Resource_Path type="string" describe="BWResource Manager source path">
        res
      </Resource_Path>    
      <Universe_Path type="string" describe="Universe(chunks) source path">
        universes/eg/jd_shuijing
      </Universe_Path>    
      <Window_HMap_size describe="Window and Height Map size">
        <Width type="int">1280</Width>
        <Height type="int">800</Height>    
      </Window_HMap_size>
      <Num_Chunks  describe="Num of the nearby chunks to consider">
        <Left type = "int">1</Left>
        <Right type = "int">1</Right>
        <Top type = "int">1</Top>
        <Bottom type = "int">1</Bottom>
      </Num_Chunks>
      <Light_Set describe="Light Settings">
        <Light> <!--- 光源1 -->
          <Pos describe="Light Pos">
            <World_X type = "double">100.0</World_X>
            <World_Y type = "double">100.0</World_Y>
            <World_Z type = "double">100.0</World_Z>
          </Pos>
          <Direction describe = "Light Direction">
            <Horizontal type = "double">3.92699081699</Horizontal>
            <Vertical type = "double">-0.78539816339</Vertical>
          </Direction>
          <ProjectionType type = "string" describe = "Directional/Point">
            Directional
          </ProjectionType>
          <Projection_Parameters>
            <Width type = "double">300.0</Width>
            <Height type = "double">300.0</Height>
            <Near type = "double">-500.0</Near>
            <Far type = "double">1500.0</Far>
          </Projection_Parameters>
        </Light>
        <Light> <!--- 光源2 -->
          <Pos describe="Light Pos">
            <World_X type = "double">0.0</World_X>
            <World_Y type = "double">100.0</World_Y>
            <World_Z type = "double">0.0</World_Z>
          </Pos>
          <Direction describe = "Light Direction">
            <Horizontal type = "double">0.78539816339</Horizontal>
            <Vertical type = "double">-0.78539816339</Vertical>
          </Direction>
          <ProjectionType type = "string" describe = "Directional/Point">
            Directional
          </ProjectionType>
          <Projection_Parameters>
            <Width type = "double">300.0</Width>
            <Height type = "double">300.0</Height>
            <Near type = "double">-500.0</Near>
            <Far type = "double">1500.0</Far>
          </Projection_Parameters>
        </Light>
      </Light_Set>
      <Stencil_Bias type = "double">
          0.0005
      </Stencil_Bias>
      <Render_Delay type = "int">
        0
      </Render_Delay> 
    </Parameters>    
  </Node> 
   ```
- **工具运行**：
	设置好配置文档之后，直接运行工具即可自动处理。在工具运行中，将会出现两个窗口，一个窗口为渲染窗口，在该窗口可以看到从光源角度出发，进行渲染的图像，其中灰色用来标记周围邻近的chunks，红色标记阴影无意义的物体，绿色标记阴影有意义物体。可以通过增大Render_Delay来更长时间的观察结果。另一个窗口为控制台窗口，这里可以看到chunk处理的结果，以及从配置文件中读取成功的参数信息。

- **运行结果**：
如上述所示，运行结果会实时显示在渲染窗口中。与此同时，检测结果还会写入到场景文件中的.chunk文件的<shadow>字段中
   ```xml
      <terrain>
          <resource>	ffffffffo.cdata/terrain	</resource>
          <shadow>	2	</shadow>
      </terrain>
   ```

	，同时对于model，还会将结果写入model下的<PrimitiveGroup>中

   ```xml
     <model>
          <resource>	objects/area/st/model/rock.model	</resource>
          <dye>
              <name>	com_tex_shiq01	</name>
              <tint>	huang	</tint>
          </dye>
          <uuid>	6055E290-4CC63BB3-1E14A3AE-0DFBE9A0	</uuid>
          <transform>
              <row0>	-0.991167 0.118751 0.820961	</row0>
              <row1>	-0.662710 0.655975 -0.894993	</row1>
              <row2>	-0.498893 -1.107288 -0.442164	</row2>
              <row3>	46.615074 -96.943344 12.009224	</row3>
          </transform>
          <affectByBlend>	true	</affectByBlend>
          <specPixelPerMeter>	20.000000	</specPixelPerMeter>
          <shadow>	2
              <RenderSet>
                  <Geometry>
                      <PrimitiveGroup>	2	</PrimitiveGroup>
                  </Geometry>
              </RenderSet>
          </shadow>
      </model>
   ```

	写入格式为32位的int格式，其中每一位代表从一个光源(Light)出发的检测结果，同时model下<shadow>字段。在读取检测信息的时候，需要按位读取，其中从右数第一位开始代表光源1到光源N，例如：仅光源01与光源02，若检测结果为光源02为有意义，01无意义，则写入<PrimitiveGroup>的结果为2，即*****0010(前面位数省略)。
而对于写入<shadow>的结果，如果同一个model存在多个RenderSet或PrimitiveGroup，则结果为其model下所有		PrimitiveGroup结果的位或。
  ```xml
     <model>
          <resource>	objects/area/zw/model/zw_mlqm01_6381.model	</resource>
          <uuid>	BCF2EFBF-46D323BC-204E259B-CB3D47BA	</uuid>
          <transform>
              <row0>	-0.327740 -0.027205 0.326609	</row0>
              <row1>	0.000000 0.461894 0.038475	</row1>
              <row2>	-0.327741 0.027205 -0.326609	</row2>
              <row3>	10.023972 -88.602753 57.472660	</row3>
          </transform>
          <affectByBlend>	true	</affectByBlend>
          <dye>
              <name>	Material #35	</name>
              <tint>	bai	</tint>
          </dye>
          <shadow>	3
              <RenderSet>
                  <Geometry>
                      <PrimitiveGroup>	2	</PrimitiveGroup>
                      <PrimitiveGroup>	3	</PrimitiveGroup>
                  </Geometry>
              </RenderSet>
          </shadow>
      </model>
  ```
