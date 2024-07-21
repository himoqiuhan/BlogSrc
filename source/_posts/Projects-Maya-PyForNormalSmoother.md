---
title: 【Maya/Python】调用OpenMaya制作平滑法线工具
date: 2023-08-03 09:27:17
tags:
 - Maya
 - Python
 - OpenMaya
 - Maya.cmds
categories: Portfolio
description: 为了制作平滑描边，在Maya内制作了一个计算顶点平滑法线信息，并转换到切线空间，最后存入顶点色的小工具
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/background4.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/mayapyicon.png
---

> 因为目前只是个人使用，所以还没有打包成一个插件，目前还是几段代码。等后续完善一下功能，再将其打包到一个插件中

> 使用的是Maya2023，基于Python3写的代码，不同Maya版本之间可能会不通用

## 作用效果

![存入顶点色RGB三通道的切线空间平滑法线信息](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903094249292.png)

## 代码源码

```python
import maya.cmds as cmds
import maya.api.OpenMaya as om
import baseWindow


class SmoothNormalProcessor(baseWindow.MainGui):
    NormalOS = om.MColorArray()
    NormalTS = om.MColorArray()

    def __init__(self, name="SmoothNormal Processor"):
        super(SmoothNormalProcessor, self).__init__(name)

    def buildUI(self):
        cmds.columnLayout(adj=True)
        cmds.button(label='Smooth Normal To VertCol', ann='Smooth Normal To VertCol', h=60, w=300, command=self.CalculateSmoothNormalToVerCol)
        cmds.button(label='Smooth Normal To Texture', ann='Smooth Normal To Texture', h=60, w=300, command=self.CalculateSmoothNormalToTexture)

    def CalculateSmoothNormalToVerCol(self, *args):
        self.NormalOS.clear()
        self.NormalTS.clear()
        self.Execute(True)

    def CalculateSmoothNormalToTexture(self, *args):
        self.NormalOS.clear()
        self.NormalTS.clear()
        self.Execute(False)

    def Execute(self, isToVertex):
        SelectionModels = cmds.ls(sl=True, l=True)
        selectList = om.MGlobal.getActiveSelectionList()

        if selectList.isEmpty():
            om.MGlobal.displayError("You Need To Select A Target")
            return
        if selectList.length() > 1:
            om.MGlobal.displayError("You Can Just Handle One Target Once")
            return

        uvSet = cmds.polyUVSet(SelectionModels[0], query=True, currentUVSet=True)
        colorSet = cmds.polyColorSet(SelectionModels[0], query=True, currentColorSet=True)
        if uvSet == None:
            om.MGlobal.displayError("You Need To Creat A UV Set For Selected Model")
            return
        else:
            uvSet = uvSet[0]
        if colorSet == None:
            om.MGlobal.displayError("You Need To Apply A Color Set For Selected Model")
            om.MGlobal.displayInfo("Method: Select Target -> Mesh Display/Apply Color")
            return
        else:
            colorSet = colorSet[0]
            print("Color Set Exist: %s" % (colorSet))

        dagPath = selectList.getDagPath(0)
        fnMesh = om.MFnMesh(dagPath)
        fnMesh.setCurrentColorSetName(colorSet)
        normals = fnMesh.getNormals()
        itVerts = om.MItMeshVertex(dagPath)
        self.NormalOS.setLength(fnMesh.numFaceVertices)
        self.NormalTS.setLength(fnMesh.numFaceVertices)

        self.CalculateSmoothNormal(itVerts, normals, colorSet)
        fnMesh.setColors(self.NormalOS, colorSet)

        itFaceVerts = om.MItMeshFaceVertex(dagPath)
        self.TransformToTangentAndStoreToVertCol(itFaceVerts, uvSet, colorSet)

        if isToVertex:
            fnMesh.setColors(self.NormalTS,colorSet)
        else:
            print("To Texture is Not Available")
            white = om.MColorArray()
            for i in range(fnMesh.numFaceVertices):
                white.append(om.MColor([1.0, 1.0, 1.0]))
            fnMesh.setColors(white, colorSet)

    def CalculateSmoothNormal(self, itVerts, normals, colorSet):
        averageNormal = om.MFloatVector()
        averageColor = om.MColor()

        while not itVerts.isDone():
            associatedNormalIndices = itVerts.getNormalIndices()
            for normalIndex in associatedNormalIndices:
                averageNormal += normals[normalIndex]
            averageNormal.normalize()

            colorIndices = itVerts.getColorIndices(colorSet)
            for colorIndex in colorIndices:
                averageColor.r = averageNormal.x
                averageColor.g = averageNormal.y
                averageColor.b = averageNormal.z
                averageColor.a = 1.0
                self.NormalOS.__setitem__(colorIndex, averageColor)
            averageNormal = om.MFloatVector(0.0, 0.0, 0.0)
            itVerts.next()
        print("ObjectSpace Smooth Normal Calculation Finished")

    def TransformToTangentAndStoreToVertCol(self, itFaceVerts, uvSetName, colorSetName):
        while not itFaceVerts.isDone():
            tangent = itFaceVerts.getTangent(uvSet=uvSetName)
            bitangent = itFaceVerts.getBinormal(uvSet=uvSetName)
            normal = itFaceVerts.getNormal()

            col = itFaceVerts.getColor(colorSetName)
            colIndex = itFaceVerts.getColorIndex(colorSet=colorSetName)

            averageColor = om.MColor()
            averageNormal = om.MVector(col[0], col[1], col[2])
            x = averageNormal * tangent
            y = averageNormal * bitangent
            z = averageNormal * normal
            averageColor.r = 0.5 * (x + 1)
            averageColor.g = 0.5 * (y + 1)
            averageColor.b = 0.5 * (z + 1)
            averageColor.a = 1

            self.NormalTS.__setitem__(colIndex, averageColor)
            itFaceVerts.next()
        print("ObjectSpace To TangentSpace Transformation Finished")

```



## 使用方法

1. 首先，需要给模型手动添加一个ColorSet，直接通过Maya自带的Apply Color即可完成（这一步是代码的Bug，但是目前我如果在代码中新建ColorSet会无法正常将信息赋到顶点色中，所以暂时需要手动解决一下）

   ![为模型Apply Color](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903095630589.png)

2. 复制以下代码到Python的代码窗口中

   ```python
   import maya.cmds as cmds
   import maya.api.OpenMaya as om
   
   class MainGui(object):
   
       def __init__(self, name='QiuH Toolkit'):
           if cmds.window(name, query=True, exists=True):
               cmds.deleteUI(name)
   
           cmds.window(name)
           self.buildUI()
           cmds.showWindow()
   
       def buildUI(self):
           print("No UI Building")
   
   
   class SmoothNormalProcessor(MainGui):
       NormalOS = om.MColorArray()
       NormalTS = om.MColorArray()
   
       def __init__(self, name="SmoothNormal Processor"):
           super(SmoothNormalProcessor, self).__init__(name)
   
       def buildUI(self):
           cmds.columnLayout(adj=True)
           cmds.button(label='Smooth Normal To VertCol', ann='Smooth Normal To VertCol', h=60, w=300, command=self.CalculateSmoothNormalToVerCol)
           cmds.button(label='Smooth Normal To Texture', ann='Smooth Normal To Texture', h=60, w=300, command=self.CalculateSmoothNormalToTexture)
   
       def CalculateSmoothNormalToVerCol(self, *args):
           self.NormalOS.clear()
           self.NormalTS.clear()
           self.Execute(True)
   
       def CalculateSmoothNormalToTexture(self, *args):
           self.NormalOS.clear()
           self.NormalTS.clear()
           self.Execute(False)
   
       def Execute(self, isToVertex):
           SelectionModels = cmds.ls(sl=True, l=True)
           selectList = om.MGlobal.getActiveSelectionList()
   
           if selectList.isEmpty():
               om.MGlobal.displayError("You Need To Select A Target")
               return
           if selectList.length() > 1:
               om.MGlobal.displayError("You Can Just Handle One Target Once")
               return
   
           uvSet = cmds.polyUVSet(SelectionModels[0], query=True, currentUVSet=True)
           colorSet = cmds.polyColorSet(SelectionModels[0], query=True, currentColorSet=True)
           if uvSet == None:
               om.MGlobal.displayError("You Need To Creat A UV Set For Selected Model")
               return
           else:
               uvSet = uvSet[0]
           if colorSet == None:
               om.MGlobal.displayError("You Need To Apply A Color Set For Selected Model")
               om.MGlobal.displayInfo("Method: Select Target -> Mesh Display/Apply Color")
               return
           else:
               colorSet = colorSet[0]
               print("Color Set Exist: %s" % (colorSet))
   
           dagPath = selectList.getDagPath(0)
           fnMesh = om.MFnMesh(dagPath)
           fnMesh.setCurrentColorSetName(colorSet)
           normals = fnMesh.getNormals()
           itVerts = om.MItMeshVertex(dagPath)
           self.NormalOS.setLength(fnMesh.numFaceVertices)
           self.NormalTS.setLength(fnMesh.numFaceVertices)
   
           self.CalculateSmoothNormal(itVerts, normals, colorSet)
           fnMesh.setColors(self.NormalOS, colorSet)
   
           itFaceVerts = om.MItMeshFaceVertex(dagPath)
           self.TransformToTangentAndStoreToVertCol(itFaceVerts, uvSet, colorSet)
   
           if isToVertex:
               fnMesh.setColors(self.NormalTS,colorSet)
           else:
               print("To Texture is Not Available")
               white = om.MColorArray()
               for i in range(fnMesh.numFaceVertices):
                   white.append(om.MColor([1.0, 1.0, 1.0]))
               fnMesh.setColors(white, colorSet)
   
       def CalculateSmoothNormal(self, itVerts, normals, colorSet):
           averageNormal = om.MFloatVector()
           averageColor = om.MColor()
   
           while not itVerts.isDone():
               associatedNormalIndices = itVerts.getNormalIndices()
               for normalIndex in associatedNormalIndices:
                   averageNormal += normals[normalIndex]
               averageNormal.normalize()
   
               colorIndices = itVerts.getColorIndices(colorSet)
               for colorIndex in colorIndices:
                   averageColor.r = averageNormal.x
                   averageColor.g = averageNormal.y
                   averageColor.b = averageNormal.z
                   averageColor.a = 1.0
                   self.NormalOS.__setitem__(colorIndex, averageColor)
               averageNormal = om.MFloatVector(0.0, 0.0, 0.0)
               itVerts.next()
           print("ObjectSpace Smooth Normal Calculation Finished")
   
       def TransformToTangentAndStoreToVertCol(self, itFaceVerts, uvSetName, colorSetName):
           while not itFaceVerts.isDone():
               tangent = itFaceVerts.getTangent(uvSet=uvSetName)
               bitangent = itFaceVerts.getBinormal(uvSet=uvSetName)
               normal = itFaceVerts.getNormal()
   
               col = itFaceVerts.getColor(colorSetName)
               colIndex = itFaceVerts.getColorIndex(colorSet=colorSetName)
   
               averageColor = om.MColor()
               averageNormal = om.MVector(col[0], col[1], col[2])
               x = averageNormal * tangent
               y = averageNormal * bitangent
               z = averageNormal * normal
               averageColor.r = 0.5 * (x + 1)
               averageColor.g = 0.5 * (y + 1)
               averageColor.b = 0.5 * (z + 1)
               averageColor.a = 1
   
               self.NormalTS.__setitem__(colIndex, averageColor)
               itFaceVerts.next()
           print("ObjectSpace To TangentSpace Transformation Finished")
   
   window = smoothNormal.SmoothNormalProcessor()
   ```

3. 按下Crtl + Enter执行代码

4. 选中模型，选择想要运行的模式（目前只确保了存入顶点色的程序的稳定性，存入贴图的版本还在完善中，暂不可用）

   ![简易窗口](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903100501179.png)

5. 等待几秒即可计算并写入完毕，后续导出的模型顶点色就带有平滑法线信息



## 代码解释(带注释版本的源码)

> 代码是在学习这位大佬源码的基础上进行一些自己的改编完成的，十分推荐大家去学习这位大佬的文章：https://zhuanlan.zhihu.com/p/538660626

```python
import maya.cmds as cmds
import maya.api.OpenMaya as om
import baseWindow


class SmoothNormalProcessor(baseWindow.MainGui):
    NormalOS = om.MColorArray()
    NormalTS = om.MColorArray()
    # 类变量，用openmaya的MColorArray类来存储对象空间和切线空间下的平滑法线

    def __init__(self, name="SmoothNormal Processor"):
        # override MainGui类中的构造函数，以SmoothNormal Processor作为窗口名称回调父类的构造函数
        super(SmoothNormalProcessor, self).__init__(name)

    def buildUI(self):
        # 窗口中UI（两个Button）的创建
        cmds.columnLayout(adj=True)
        cmds.button(label='Smooth Normal To VertCol', ann='Smooth Normal To VertCol', h=60, w=300, command=self.CalculateSmoothNormalToVerCol)
        cmds.button(label='Smooth Normal To Texture', ann='Smooth Normal To Texture', h=60, w=300, command=self.CalculateSmoothNormalToTexture)

    def CalculateSmoothNormalToVerCol(self, *args):
        self.NormalOS.clear()
        self.NormalTS.clear()
        self.Execute(True)

    def CalculateSmoothNormalToTexture(self, *args):
        # 先清空两个类变量，然后再执行程序
        self.NormalOS.clear()
        self.NormalTS.clear()
        self.Execute(False)

    def Execute(self, isToVertex):
        SelectionModels = cmds.ls(sl=True, l=True)
        selectList = om.MGlobal.getActiveSelectionList()
        # Return an MSelectionList containing the nodes, components and plugs currently selected in Maya
    	# 返回的是一个选择列表MSelectionList，MSelectionList是一个MObject、MPlug、MDagPath的异构列表
    	# 需要通过遍历选择列表，通过利用遍历的index获取DAGPath，然后利用DAGPath来获取到MFnMesh
    	# MFnMesh类中存储了许多网格体的信息

        # 限制同时处理的对象（有且只能有1个）
        if selectList.isEmpty():
            om.MGlobal.displayError("You Need To Select A Target")
            return
        if selectList.length() > 1:
            om.MGlobal.displayError("You Can Just Handle One Target Once")
            return

        # 检查选中的对象是否有Colorset和UVset
        uvSet = cmds.polyUVSet(SelectionModels[0], query=True, currentUVSet=True)
        colorSet = cmds.polyColorSet(SelectionModels[0], query=True, currentColorSet=True)
        if uvSet == None:
            om.MGlobal.displayError("You Need To Creat A UV Set For Selected Model")
            return
        else:
            uvSet = uvSet[0]
        if colorSet == None:
            om.MGlobal.displayError("You Need To Apply A Color Set For Selected Model")
            om.MGlobal.displayInfo("Method: Select Target -> Mesh Display/Apply Color")
            return
        else:
            colorSet = colorSet[0]
            print("Color Set Exist: %s" % (colorSet))

        dagPath = selectList.getDagPath(0)
        # DAG是Directed Acyclic Graph的缩写
    	# DAG节点是Maya场景中的节点，包括transform和shape两种类型的节点。
    	# transform节点提供位置、旋转、缩放等信息，并且可以有子节点；shape节点只提供几何信息，并且没有子节点。
    	# 可以把dagPath理解为指向一个特定模型对象的路径，通过这个路径可以获取有关这个物体的很多信息，并以此填充到特定数据结构（类）中
        fnMesh = om.MFnMesh(dagPath)
        # 通过dagPath路径获得多边形网格模型
        fnMesh.setCurrentColorSetName(colorSet)
        normals = fnMesh.getNormals()
        # Returns a copy of the mesh's normals. The normals are the per-polygon per-vertex normals.
    	# 即获取模型的法线信息，返回值是MFloatVectorArray，如果要获得特定顶点的法线，需要通过getFaceNormalIds()来获得数组的索引
        itVerts = om.MItMeshVertex(dagPath)
        # 获取多边形网格模型的顶点迭代器vertex iterator
        self.NormalOS.setLength(fnMesh.numFaceVertices)
        self.NormalTS.setLength(fnMesh.numFaceVertices)
        # 基于顶点数量来设置之前创建的数组的长度

        # 因为只有Vertex才能读取到与顶点相关联的法线的索引，所以由vertex来计算模型空间下的平滑法线信息：
        self.CalculateSmoothNormal(itVerts, normals, colorSet)
        fnMesh.setColors(self.NormalOS, colorSet)
        # 通过遍历顶点来计算模型空间下的顶点平滑法线，并写入顶点色中

        # 因为只有FaceVertex才能读取到切线、副切线的信息，所以由FaceVertex来将模型空间下的法线信息转换到切线空间：
        itFaceVerts = om.MItMeshFaceVertex(dagPath)
        # 获取FaceVertex的迭代器
        self.TransformToTangentAndStoreToVertCol(itFaceVerts, uvSet, colorSet)
		# 将之前计算好的平滑法线信息转换到切线空间中，并最终存储到顶点色的RGB通道中
        
        # 区分是将结果写入顶点色还是写入贴图
        if isToVertex:
            fnMesh.setColors(self.NormalTS,colorSet)
            # 设置顶点色
        else:
            print("To Texture is Not Available")
            white = om.MColorArray()
            for i in range(fnMesh.numFaceVertices):
                white.append(om.MColor([1.0, 1.0, 1.0]))
            fnMesh.setColors(white, colorSet)

    def CalculateSmoothNormal(self, itVerts, normals, colorSet):
        averageNormal = om.MFloatVector()
        averageColor = om.MColor()

        while not itVerts.isDone():
            # 逐个顶点进行计算和顶点色写入
            associatedNormalIndices = itVerts.getNormalIndices()
            # This method returns the normal indices of the face/vertex associated with the current vertex
        	# 返回与这个顶点相关联的顶点、面的法线索引（平均值这不仅来了吗！）
            for normalIndex in associatedNormalIndices:
                averageNormal += normals[normalIndex]
            averageNormal.normalize()
            # 求和取平均，直接使用MFloatVector类带的normalize即可

            colorIndices = itVerts.getColorIndices(colorSet)
            # This method returns the colorIndices into the color array
        	# 获取当前顶点的全部顶点色索引（一个顶点可能有多个顶点色索引，指向不同的颜色集合，这一点我认为是和一个顶点在Maya中可能有多个法线同理，导出后会变成多个顶点，但会与信息一一对应）
            for colorIndex in colorIndices:
                averageColor.r = averageNormal.x
                averageColor.g = averageNormal.y
                averageColor.b = averageNormal.z
                averageColor.a = 1.0
                self.NormalOS.__setitem__(colorIndex, averageColor)
                # 写入NormalOS的Color数组，因为这套逻辑中设置顶点是直接统一用一个数组去Copy设置的，所以需要保证这里的索引值与顶点色中的索引值相同
            	# 同样也可以一个一个去setColors
            averageNormal = om.MFloatVector(0.0, 0.0, 0.0)
            # 重置一下averageNormal，然后继续计算下一个平滑法线颜色
            itVerts.next()
        print("ObjectSpace Smooth Normal Calculation Finished")

    def TransformToTangentAndStoreToVertCol(self, itFaceVerts, uvSetName, colorSetName):
        while not itFaceVerts.isDone():
            # 使用面顶点迭代器遍历每个面顶点
            tangent = itFaceVerts.getTangent(uvSet=uvSetName)
            bitangent = itFaceVerts.getBinormal(uvSet=uvSetName)
            normal = itFaceVerts.getNormal()
			# 得到当前面顶点的切线和副切线、法线，用于构建TBN矩阵（只有FaceVertex才能获取到切线信息）
            col = itFaceVerts.getColor(colorSetName)
            colIndex = itFaceVerts.getColorIndex(colorSet=colorSetName)
            # 得到当前 面顶点的顶点色和顶点色索引，用于读取之前计算出的模型空间下的平滑法线信息

            # 左乘TBN矩阵，将法线信息从模型空间转换到切线空间
        	# （   tangent   ）    normal.x
        	# （  bitangent  ） *  normal.y
        	# （    normal   ）    normal.z
            averageColor = om.MColor()
            averageNormal = om.MVector(col[0], col[1], col[2])
            x = averageNormal * tangent
            y = averageNormal * bitangent
            z = averageNormal * normal
            # 因为计算出来的法线信息是在[-1,1]区间上的，而颜色信息是在[0,1]区间上的，所以存储前需要进行一次映射
            averageColor.r = 0.5 * (x + 1)
            averageColor.g = 0.5 * (y + 1)
            averageColor.b = 0.5 * (z + 1)
            averageColor.a = 1

            self.NormalTS.__setitem__(colIndex, averageColor)
            itFaceVerts.next()
        print("ObjectSpace To TangentSpace Transformation Finished")

```



## 学习心得

起初看了一些写Python代码的教程，是通过学习Python调用Maya.cmds来入门的Python For Maya，也跟着教程简单的写了几个小工具。到后面实际去做自己想要的工具时，发现cmds能够实现的功能很有限制，同时找到了那位大佬的文章，就去跟着他的代码学习了一下OpenMaya的库，基本上就是结合它的代码，一行一行地读，不会的就去官方文档查，并借助NewBing去寻找更多的资料。在全部理解完大佬的代码后，就参考着他的源码，结合之前学的一些创建可视化窗口和面向对象的知识，写出了这个工具



## 学习资料

> 只列出了一些重点资料，更多的知识自己基于需求去查询和学习

【python+maya 脚本语法编写全面基础入门中文字幕视频教程】 https://www.bilibili.com/video/BV1d4411a7aH

【Maya工作流的平滑法线描边小工具 - 鲜虾云吞面的文章】 https://zhuanlan.zhihu.com/p/538660626

【Maya官方API文档 - OpenMaya】 https://help.autodesk.com/view/MAYAUL/2023/CHS/?guid=MAYA_API_REF_py_ref_namespace_open_maya_html

【Maya官方API文档 - PyForMaya】 https://help.autodesk.com/view/MAYAUL/2023/CHS/?guid=__CommandsPython_index_html
