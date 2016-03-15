package require vtk
package require vtkinteraction

vtkRenderer ren1
ren1 SetBackground 0.8 0.8 0.8
vtkRenderWindow renWin
renWin AddRenderer ren1
renWin SetSize 400 300

vtkRenderWindowInteractor iren
iren SetRenderWindow renWin
iren Initialize; # start the interaction

vtkLookupTable clut
clut SetHueRange 0.0 0.67
clut SetAlphaRange 0.2 0.9
clut Build

vtkSliderRepresentation2D sliderRep
  sliderRep SetMinimumValue 0
  sliderRep SetMaximumValue 150
  sliderRep SetValue 0
  sliderRep SetTitleText "Iso Value/ Planar Cross Section"
  [sliderRep GetPoint1Coordinate] SetCoordinateSystemToNormalizedDisplay
    [sliderRep GetPoint1Coordinate] SetValue 0.3 0.1
    [sliderRep GetPoint2Coordinate] SetCoordinateSystemToNormalizedDisplay
    [sliderRep GetPoint2Coordinate] SetValue 0.9 0.1

  # Specifiy the sizes of the slider, the end caps and the slider's tube.
  #
    sliderRep SetSliderLength 0.02
    sliderRep SetSliderWidth 0.03
    sliderRep SetEndCapLength 0.01
    sliderRep SetEndCapWidth 0.03
    sliderRep SetTubeWidth 0.005
    sliderRep SetLabelFormat "%4.0lf"

  vtkSliderWidget sliderWidget
    sliderWidget SetInteractor iren
    sliderWidget SetRepresentation sliderRep
    sliderWidget KeyPressActivationOff

sliderWidget SetAnimationModeToAnimate
sliderWidget AddObserver InteractionEvent callback

vtkDataSetReader reader
reader SetFileName "ironProt.vtk"
reader Update
set range [[reader GetOutput] GetScalarRange]

vtkOutlineFilter outline
outline SetInput [reader GetOutput]
vtkPolyDataMapper outlineMapper
outlineMapper SetInput [outline GetOutput]
vtkActor outlineActor
outlineActor SetMapper outlineMapper
set outlineProp [outlineActor GetProperty]
eval $outlineProp SetColor 0 0 0
ren1 AddActor outlineActor

vtkContourFilter iso
#vtkMarchingCubes iso
iso SetInput [reader GetOutput]
iso SetValue 0 [expr ([lindex $range 1] - [lindex $range 0]) * 0.9 + [lindex $range 0]]
#iso ComputeGradientsOn

vtkDataSetMapper isoMapper
isoMapper SetInput [iso GetOutput]
isoMapper ScalarVisibilityOn
eval isoMapper SetScalarRange $range
isoMapper ImmediateModeRenderingOn
isoMapper SetLookupTable clut
vtkActor isoActor
isoActor SetMapper isoMapper
ren1 AddActor isoActor

vtkPlane plane
set bounds [[reader GetOutput] GetBounds]
set zval [expr ([lindex $bounds 5] - [lindex $bounds 4]) * (50.0 / 100.0) + [lindex $bounds 4]]
plane SetOrigin 0 0 $zval
plane SetNormal 0 0 1
vtkCutter slicer
slicer SetInput [reader GetOutput]
slicer SetCutFunction plane
vtkPolyDataMapper sliceMapper
sliceMapper SetInput [slicer GetOutput]
sliceMapper SetLookupTable clut
eval sliceMapper SetScalarRange $range
vtkActor sliceActor
sliceActor SetMapper sliceMapper
ren1 AddActor sliceActor
ren1 Render

sliderWidget On
iren AddObserver UserEvent {wm deiconify .vtkInteract}
wm withdraw .
vtkMath math

# Callback for the interaction
#
proc callback { } {
iso SetValue 0 [ eval math Round [ sliderRep GetValue ] ]
plane SetOrigin 0 0 [ eval math Round [ sliderRep GetValue ] ]
}

iren Start
