# 3DSMagic
Random collection of 3D Slicer Magic Spells

```
### Convert all labelmaps to models

#create segmentation node
emptyVolumeNode = slicer.mrmlScene.AddNewNodeByClass("vtkMRMLScalarVolumeNode", "output volume")
segmentationNode = slicer.mrmlScene.AddNewNodeByClass("vtkMRMLSegmentationNode")
segmentationNode.CreateDefaultDisplayNodes()
segmentationNode. SetReferenceImageGeometryParameterFromVolumeNode(emptyVolumeNode)

#add all loaded label nodes as segments with the same name
label_nodes = getNodesByClass('vtkMRMLLabelMapVolumeNode')
for labelmapVolumeNode in label_nodes:
	#create empty segment with correct name
	segmentId = labelmapVolumeNode.GetName().replace('msk_tra-label','') # change this accordingly
	segmentIds = vtk.vtkStringArray()
	segmentIds.InsertNextValue(segmentId)
	segmentationNode.GetSegmentation().AddEmptySegment(segmentId)
	#update the segment
	slicer.modules.segmentations.logic().ImportLabelmapToSegmentationNode(labelmapVolumeNode, segmentationNode, segmentIds)

#folder name (model hierarchy)
shNode = slicer.mrmlScene.GetSubjectHierarchyNode()
exportFolderItemId = shNode.CreateFolderItem(shNode.GetSceneItemID(), "PAT")
#convert all segments to models
slicer.modules.segmentations.logic().ExportSegmentsToModels(segmentationNode,segmentationNode.GetSegmentation().GetSegmentIDs(),exportFolderItemId)
```
## Capture models with black background
```
outdir = 'D:/2023/trachea/png/';
viewNodeID = 'vtkMRMLSliceNodeRed'
import ScreenCapture
cap = ScreenCapture.ScreenCaptureLogic()
view = slicer.app.layoutManager().threeDWidget(0).threeDView()
view.mrmlViewNode().SetBackgroundColor(0,0,0)
view.mrmlViewNode().SetBackgroundColor2(0,0,0)
view.mrmlViewNode().SetAxisLabelsVisible(False)
view.mrmlViewNode().SetBoxVisible(False)


nodes = [node for node in getNodesByClass('vtkMRMLModelNode') if "AKP" in node.GetName() or "PAT" in node.GetName()]
for node in nodes:
	node.GetDisplayNode().SetVisibility(False)
	node.GetDisplayNode().SetColor(1,1,1)

for node in nodes:
	node.GetDisplayNode().SetVisibility(True)
	view.resetFocalPoint()
	#view.resetCamera()
	view.forceRender()
	cap.captureImageFromView(view, outdir+node.GetName()+'.png')
	node.GetDisplayNode().SetVisibility(False)
```
