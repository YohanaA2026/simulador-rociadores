import math
from pathlib import Path
import pandas as pd
import streamlit as st

st.set_page_config(page_title='Simulador de rociadores | Parcial', layout='wide', page_icon='💧')
st.title('💧 Simulador interactivo de rociadores')
st.caption('Basado en el ejercicio 3 del taller: nave de 45 × 24 m, área desfavorable de 216 m², SCH40, K80.')
st.warning('Modelo académico de predimensionamiento: NO representa un equilibrado hidráulico completo de la parrilla. La presión de bomba es una estimación; verifica conexiones, pérdidas locales, topología y criterios normativos antes de usarla como resultado final.')

with st.sidebar:
 st.header('Datos editables')
 largo=st.number_input('Largo del edificio (m)',1.,1000.,45.,1.)
 ancho=st.number_input('Ancho del edificio (m)',1.,1000.,24.,1.)
 densidad=st.number_input('Densidad (L/min/m²)',0.1,100.,5.,0.5)
 area=st.number_input('Área de diseño (m²)',1.,10000.,216.,9.)
 area_rociador=st.number_input('Cobertura por rociador (m²)',1.,50.,9.,1.)
 k=st.number_input('Factor K (L/min/√bar)',1.,500.,80.,5.)
 p_min=st.number_input('Presión mínima por rociador (bar) — supuesto',0.1,20.,0.5,0.1)
 C=st.number_input('Hazen–Williams C (acero negro; verificar)',50.,160.,120.,5.)
 altura=st.number_input('Desnivel entre bomba y rociadores (m)',0.,100.,3.6,0.1)
 n_filas=st.number_input('Ramales activos en el área desfavorable',1,30,4,1)
 n_por_ramal=st.number_input('Rociadores activos por ramal',1,30,6,1)
 longitud=st.number_input('Separación entre rociadores (m)',0.1,30.,3.,0.5)
 d_ramal=st.number_input('Diámetro interior ramal SCH40 (mm) — verificar',5.,300.,40.89,0.1)
 d_colector=st.number_input('Diámetro interior colector 3 in SCH40 (mm)',5.,300.,77.93,0.1)
 d_principal=st.number_input('Diámetro interior principal 4 in SCH40 (mm)',5.,300.,102.26,0.1)
 l_colector=st.number_input('Longitud por sección de colector (m)',0.1,100.,3.,0.5)
 l_principal=st.number_input('Longitud alimentación principal (m) — supuesto',0.,300.,10.,1.)
 reserva=st.number_input('Autonomía reserva (min) — supuesto',1,240,60,5)

st.subheader('Plano del ejercicio y área desfavorable')
col1,col2=st.columns([1.1,1])
with col1:
 st.image(str(Path(__file__).parent/'plano_ejercicio3.png'),caption='Plano original del ejercicio 3 (zona verde = área desfavorable)',use_container_width=True)
with col2:
 n_ideal=math.ceil(area/area_rociador)
 n=n_filas*n_por_ramal
 q_densidad=densidad*area
 q_min_unit=densidad*area_rociador
 q_unit=max(q_min_unit,k*math.sqrt(p_min))
 q_total=q_unit*n
 p_unit=(q_unit/k)**2
 st.metric('Caudal mínimo por densidad',f'{q_densidad:,.1f} L/min')
 st.metric('Rociadores teóricos (área/cobertura)',f'{n_ideal}')
 st.metric('Rociadores modelados',f'{n}')
 st.metric('Caudal inicial del modelo',f'{q_total:,.1f} L/min')
 if n!=n_ideal: st.error('El número de rociadores modelados no coincide con área/cobertura. Ajusta filas y rociadores por ramal.')
 if q_total<q_densidad: st.error('El caudal inicial no alcanza el mínimo de densidad.')
 st.info('El caudal de cada rociador se fija como valor inicial uniforme. En un cálculo hidráulico completo, cada descarga varía con su presión local.')

# Hazen Williams in SI: head loss m, Q m³/s, d m
def perdida_bar(q_lmin, length_m, diameter_mm, c):
 if q_lmin<=0 or length_m<=0: return 0.
 q=q_lmin/60000.
 d=diameter_mm/1000.
 hf=10.67*length_m*(q**1.852)/(c**1.852*d**4.871)
 return hf*0.0980665

st.subheader('Cálculo detallado: recorrido crítico simplificado')
st.write('Se calcula un ramal desde el rociador más alejado hacia su colector. Después se suman las pérdidas de una trayectoria representativa por el colector y la alimentación principal. Las pérdidas se calculan con Hazen–Williams.')
rows=[]
p=p_unit
for i in range(1,n_por_ramal+1):
 qtr=q_unit*i
 dp=perdida_bar(qtr,longitud,d_ramal,C)
 p+=dp
 rows.append({'Tramo':f'Ramal {i}','Caudal (L/min)':qtr,'Longitud (m)':longitud,'Diámetro int. (mm)':d_ramal,'Pérdida (bar)':dp,'Presión acumulada (bar)':p})
for i in range(1,n_filas+1):
 qtr=q_unit*n_por_ramal*i
 dp=perdida_bar(qtr,l_colector,d_colector,C)
 p+=dp
 rows.append({'Tramo':f'Colector {i}','Caudal (L/min)':qtr,'Longitud (m)':l_colector,'Diámetro int. (mm)':d_colector,'Pérdida (bar)':dp,'Presión acumulada (bar)':p})
dp=perdida_bar(q_total,l_principal,d_principal,C)
p+=dp
rows.append({'Tramo':'Alimentación principal','Caudal (L/min)':q_total,'Longitud (m)':l_principal,'Diámetro int. (mm)':d_principal,'Pérdida (bar)':dp,'Presión acumulada (bar)':p})
fric=sum(r['Pérdida (bar)'] for r in rows)
pelev=altura*0.0980665
p_est=p+pelev
st.dataframe(pd.DataFrame(rows).style.format({'Caudal (L/min)':'{:,.2f}','Longitud (m)':'{:.2f}','Diámetro int. (mm)':'{:.2f}','Pérdida (bar)':'{:.5f}','Presión acumulada (bar)':'{:.5f}'}),use_container_width=True,hide_index=True)

st.subheader('Resultados de predimensionamiento')
a,b,c,d=st.columns(4)
a.metric('Caudal modelado',f'{q_total:,.1f} L/min')
b.metric('Pérdidas por fricción',f'{fric:.3f} bar')
c.metric('Presión bomba estimada',f'{p_est:.3f} bar')
d.metric('Reserva orientativa',f'{q_total*reserva/1000:,.1f} m³')
st.caption('Presión estimada = presión del rociador inicial + fricción en trayectoria simplificada + desnivel. No incluye accesorios, válvulas, curvas de bomba ni equilibrado de presiones entre ramales.')
with st.expander('Fórmulas y conversiones usadas',expanded=False):
 st.latex(r'Q_{diseño}=D\cdot A')
 st.latex(r'Q_{rociador}=K\sqrt{P_{rociador}}')
 st.latex(r'h_f=10.67\frac{L\,Q^{1.852}}{C^{1.852}\,d^{4.871}}')
 st.write('Para Hazen–Williams: Q en m³/s, d en m, L y hf en m; 1 mca ≈ 0,0980665 bar.')
 st.latex(r'P_{bomba,est}=P_{rociador}+\sum\Delta P_f+0.0980665\,\Delta z')
 st.write('Las dimensiones interiores SCH40 son valores de referencia editables, no diámetros nominales.')

csv=pd.DataFrame(rows).to_csv(index=False).encode('utf-8-sig')
st.download_button('⬇️ Exportar tabla de cálculo (CSV)',csv,'calculo_rociadores.csv','text/csv')
st.divider()
st.markdown('**Para un resultado definitivo:** es necesario definir la conectividad exacta de la parrilla y resolver los caudales/presiones en cada nodo de manera simultánea. La figura del taller permite ver el trazado, pero no fija todas las longitudes de conexión a bomba ni las pérdidas locales.')
