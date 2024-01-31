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

## Assign ctrl+T to load next volume and matching labelmap

```
import glob, os

niigz_indir = '/path/to/ct_datadir_niigz'
niigzs = sorted([os.path.basename(fname).replace('.nii.gz','') for fname in glob.glob(os.path.join(niigz_indir,'*.nii.gz'))])
label_indir = '/path/to/labeldir_niigz'

tmps = []
out_fnames = []

def load():
	global volume, label
	for i, pat in enumerate(niigzs):
		print(pat, i+1, 'of', len(niigzs))
		fname = os.path.join(niigz_indir,pat+'.nii.gz')
		label_fname = os.path.join(label_indir,pat+'-label.nii.gz')
		if 'volume' in globals() and slicer.mrmlScene.IsNodePresent(volume):
			slicer.mrmlScene.RemoveNode(volume)
		if 'label' in globals() and slicer.mrmlScene.IsNodePresent(label):
			slicer.mrmlScene.RemoveNode(label)

		volume= slicer.util.loadVolume(fname)
		label = slicer.util.loadLabelVolume(label_fname)
		
		layoutmanager = slicer.app.layoutManager()
		for sliceViewName in layoutmanager.sliceViewNames():
			sliceNode = layoutmanager.sliceWidget(sliceViewName).sliceView().mrmlSliceNode()
			compositeNode = slicer.app.applicationLogic().GetSliceLogic(sliceNode).GetSliceCompositeNode()
			compositeNode.SetLabelOpacity(0.6)			
		
		yield pat

l = load()
qt.QShortcut(qt.QKeySequence("Ctrl+T"), slicer.util.mainWindow()).activated.connect(lambda: next(l))
```
