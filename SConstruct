from rsf.proj import *
import rsf.recipes.fdmod as fdmod

# Thanks to Ning Tu
par = par = {
    'nx':401,  'ox':2, 'dx':5,     'lx':'x', 'ux':'m',
    'nz':401,  'oz':2, 'dz':5,     'lz':'z', 'uz':'m',
    'nt':1000,  'ot':0, 'dt':0.0005, 'lt':'t', 'ut':'s',
    'kt':200,
    'uk':'1/m',
    'jsnap':5,
    'nb':300,
    'frq':20,
    'nbell':5,
    'snapt':0.5
}

fdmod.param(par)
par['nt'] += par['kt']

# Velocity
Flow('vel', None,
    '''
    math
        n1=%(nz)g d1=%(dz)g o1=%(oz)g unit1=m
        n2=%(nx)g d2=%(dx)g o2=%(ox)g unit2=m
        output="1500+1*x1"
    ''' % par)
Plot('vel',
     '''
     grey wanttitle=n scalebar=y allpos=y bias=1500
     barlabel=Velocity barunit=m/s barreverse=y
     ''')

# Density
Flow('den','vel','math output=1')

# Source
#Flow('sou',None,
     #'spike n1=2 nsp=2 k1=1,2 mag=1000,500 o1=0 o2=0')
(sx, sz) = (1000, 500)
Flow('sou', None,
     'spike n1=2 nsp=2 k1=1,2 mag=%g,%g o1=0 o2=0' % (sx, sz)
     )

Plot('sou',
     '''
     dd type=complex |
     graph symbol=* symbolsz=10 wantaxis=n
     plotcol=3 plotfat=3 wanttitle=n scalebar=y
     yreverse=y
     ''')

# Receiver
Flow('rec1', None,
     'spike n1=2 nsp=2 k1=1,2 mag=%g,%g o1=0 o2=0' % (913, 1500)
     )
Flow('rec2', None,
     'spike n1=2 nsp=2 k1=1,2 mag=%g,%g o1=0 o2=0' % (1087, 1500)
     )
Flow('rec', ['rec1', 'rec2'], 'cat axis=2 ${SOURCES[1]} ')

Plot('rec',
     '''
     dd type=complex |
     graph symbol=o symbolsz=10 wantaxis=n
     plotcol=3 plotfat=3 wanttitle=n scalebar=y
     yreverse=y
     ''')

Result('vel','vel sou rec','Overlay')

# Wavelet
Flow('wav', None,
    '''
    spike nsp=1 n1=%(nt)g d1=%(dt)g k1=%(kt)d |
    ricker1 frequency=%(frq)g |
    transp
    ''' % par
    )
Result('wav','wiggle poly=y pclip=100 title=Wavelet')

# Finite-difference modeling
Flow('predat wvl','wav vel den sou rec',
     '''
     awefd2d vel=${SOURCES[1]} den=${SOURCES[2]} 
     sou=${SOURCES[3]} rec=${SOURCES[4]} wfl=${TARGETS[1]} 
     verb=y free=n snap=y dabc=y jdata=1 jsnap=%(jsnap)g
     ''' % par)

par['nt'] -= par['kt']
Flow('dat', 'predat',
    '''
    transp |
    window f1=%(kt)g j1=1 squeeze=n |
    put o1=0
    ''' % par)

# Run 'scons wvl.vpl' to watch a movie
Plot('wvl',
     '''
     grey gainpanel=a title=Wavefield screenratio=1
     label1=Depth label2=Distance unit1=m unit2=m
     ''',view=1)

Result('wvl',
       '''
       window n3=1 min3=0.4 |
       grey title="Wave at 0.4 s" screenratio=1
       label1=Depth label2=Distance unit1=m unit2=m
       ''')

End()
