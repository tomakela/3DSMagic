# 3DSMagic
Random collection of 3D Slicer Magic Spells

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

