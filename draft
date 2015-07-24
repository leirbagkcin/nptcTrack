import numpy as np
import cv2
import pyqtgraph as pg
from pyqtgraph.Qt import QtCore, QtGui
import RawImageWidget as rw
from pyqtgraph.dockarea import *



cap= cv2.VideoCapture('/home/ngabriel/Desktop/trackAuto/3_2.avi')

app = QtGui.QApplication([])

win = QtGui.QMainWindow()
area = DockArea()
win.setCentralWidget(area)

w=cap.get(3)
h=cap.get(4)

print w,h

win.resize(w+670,h+50)
win.setWindowTitle('PORTrack')



d1 = Dock("Dock1", size = (w+20,h))
d2 = Dock("Dock2", size = (650,h+50))


area.addDock(d1, 'left')
area.addDock(d2, 'right')


w1 = pg.LayoutWidget()

btnSt=QtGui.QPushButton('Reset Center')
btnEn=QtGui.QPushButton('End')

p5 = pg.PlotWidget()
p5.showGrid(x=True, y=True)

p2 = rw.RawImageWidget()

d1.hideTitleBar()
h3 = pg.LayoutWidget()
h3.addWidget(btnSt, row=0, col=0)
h3.addWidget(btnEn, row=0, col=1)
w1.addWidget(h3, row=1, col=0)
w1.addWidget(p2, row=0, col=0)


d1.addWidget(w1)

w2 = pg.LayoutWidget()


d2.hideTitleBar()
w2.addWidget(p5)
d2.addWidget(w2)

win.show()


while(True):
    
    n=0

    while n < 1:

        _, frame = cap.read()
        gy = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

        # setup blob detector parameters
        params = cv2.SimpleBlobDetector_Params()

        # define threshold parameter search space
        params.minThreshold = 65
        params.maxThreshold = 125
        
        params.thresholdStep = 5

        #filter blobs by area
        params.filterByArea = True
        params.minArea = 800
        params.maxArea = 5000

        try:
            #initialize blob detectors with specified parameters
            det = cv2.SimpleBlobDetector(params)

            bl = det.detect(gy)
            #get pupil x and y coordinates (center of blob)
            p = bl[0].pt
            #get pupil radius
            s = bl[0].size

            # x,y,r must be integers for circle plotting
            x0 = int(round(p[0]))
            y0 = int(round(p[1]))
            r0 = int(round(s))
            # set n=1 to exit while loop
            n+=1
            
        except:
            n=0
            
    def Start():

        global x0,y0,r0
        
        n=0

        while n < 1:

            _, frame = cap.read()
            gy = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
            

            # setup blob detector parameters
            params = cv2.SimpleBlobDetector_Params()

            # define threshold parameter search space == min,min+Step,...,max-Step
            params.minThreshold = 70
            params.maxThreshold = 125
            
            params.thresholdStep = 5

            #filter blobs by area
            params.filterByArea = True
            params.minArea = 1500
            params.maxArea = 5000

            try:
                #initialize blob detectors with specified parameters
                det = cv2.SimpleBlobDetector(params)

                bl = det.detect(gy)
                p = bl[0].pt
                s = bl[0].size

                x0 = int(round(p[0]))
                y0 = int(round(p[1]))
                r0 = int(round(s))
                n+=1
                
            except:
                n=0

    btnSt.clicked.connect(Start)


    def End():
        cap.release()
        import sys
        sys.exit()

    btnEn.clicked.connect(End)
    
    def update1():

        global dv,p1
        
        n=0
        _,frame = cap.read()
        gy = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        _,bn = cv2.threshold(gy,95,500,cv2.THRESH_BINARY)



        try:

            det = cv2.SimpleBlobDetector(params)

            bl = det.detect(gy)
            p = bl[0].pt
            s = bl[0].size

            x = int(round(p[0]))
            y = int(round(p[1]))
            r = int(round(s))


            dv = (((p[0]-x0)**2) + ((p[1]-y0)**2))**.5

            #Dv.append(dv)

            

            #circle for first frame
            cv2.circle(frame,(x0,y0),r0,(255,255,255),1)
            #circle for current frame
            cv2.circle(frame,(x,y),r0,(255,0,0),1)
            #center of circle
            cv2.circle(frame,(x,y),2,(255,0,0),1)

            
            
        except:
            cv2.circle(frame,(x0,y0),r0,(255,255,255),1)
            
        frame = cv2.transpose(frame)
        
        p1=p2.setImage(frame)
    
        #cv2.imshow('binary',bn)

        
    # add one new plot curve for every 100 samples
    chunkSize = 100
    # Remove chunks after we have 10
    maxChunks = 10
    startTime = pg.ptime.time()

    p5.setLabel('bottom', 'Time', 's')
    p5.setLabel('left', 'variance', 'millimeters')
    p5.setXRange(-10, 0)
    p5.setYRange(0,4)
    curves = []
    data5 = np.empty((chunkSize+1,2))
    ptr5 = 0

    def update3():
        global p5, data5, ptr5, curves
        
        now = pg.ptime.time()
        for c in curves:
            c.setPos(-(now-startTime), 0)
        
        i = ptr5 % chunkSize
        if i == 0:
            curve = p5.plot()
            curves.append(curve)
            last = data5[-1]
            data5 = np.empty((chunkSize+1,2))        
            data5[0] = last
            while len(curves) > maxChunks:
                c = curves.pop(0)
                p5.removeItem(c)
        else:
            curve = curves[-1]
        
        data5[i+1,0] = now - startTime
        try:
            data5[i+1,1] = 0.09375*(dv)
        except:
            pass
        
        curve.setData(x=data5[:i+2, 0], y=data5[:i+2, 1])
        ptr5 += 1


    # update all plots
    def update():

        update1()
        update3()
        
    timer = pg.QtCore.QTimer()
    timer.timeout.connect(update)
    timer.start(1)



    ## Start Qt event loop unless running in interactive mode or using pyside.
    if __name__ == '__main__':
        import sys
        if (sys.flags.interactive != 1) or not hasattr(QtCore, 'PYQT_VERSION'):
            
            QtGui.QApplication.instance().exec_()
