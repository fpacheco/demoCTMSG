#!/usr/bin/python
"""
Sencilla prueba de -html- scrapy con lxml realizado para CTMSG
Autor: Fernando Pacheco (Ingesur SRL) <fernando.pacheco@ingesur.com.uy> 
"""
# Para scarpy
from lxml import html
# SQlite 3
import sqlite3
# Standard logging module
import logging
# Para parsear del documento
import urllib2 
# para la hora del logging
import datetime

LOG_FILENAME = '/home/fpacheco/trabajo/computacion/demoCTMSG/logging.log'
# Por ahora no! 
#URL_EMBALSE_PASSO_FUNDO = "http://www.tractebelenergia.com.br/modules/system/viewPage.asp?P=849&VID=default&SID=683605934420294&S=1&template=popup&A=open:nivel:tabela_usina:UHPF&C=1505"
# Datos de lluvia en una estacion
LLUVIA_PASSO_FUNDO = "http://www.tractebelenergia.com.br/telemetria/telemetria/telemetricas_passofundo.html"
# Base de datos de SQLite 3 elemental
DATABASE ="/home/fpacheco/trabajo/computacion/demoCTMSG/demoPF.sqlite3"
# Por ahora no!
# CREATE TABLE pf(fecha text, hora text,nivel real,volumen real,qa real,qe real);

# La tabla
# CREATE TABLE lpf(fecha text,hora text,pBF real,pPF real);

# Niveles de logeo
LOGLEVELS = {'debug': logging.DEBUG,
             'info': logging.INFO,
             'warning': logging.WARNING,
             'error': logging.ERROR,
             'critical': logging.CRITICAL}

def guardaLluviaPF(data):
  conn = sqlite3.connect(DATABASE)
  c = conn.cursor()
  for d in data:
    strSQL = "SELECT * FROM lpf WHERE fecha='%s' AND hora='%s' AND pBF=%f AND pPF=%f;" % (d['fecha'],d['hora'],d['pBF'],d['pPF'])
    c.execute(strSQL)
    if c.fetchone() == None:
      strSQL = "INSERT INTO lpf VALUES('%s','%s',%f,%f);" % (d['fecha'],d['hora'],d['pBF'],d['pPF'])
      #print strSQL
      c.execute(strSQL)  
  conn.commit()
  c.close()

def obtieneDatos(url):
  data = []
  doc = html.fromstring(urllib2.urlopen(LLUVIA_PASSO_FUNDO).read())
  # De la tabla de clase "texto" tomo todas las filas
  filas =  doc.cssselect('table.texto tr')
  # Las 2 ultimas no interesan
  nfilas = len(filas)
  # y en la fila 3 comienzan los datos
  for tr in filas[3:(nfilas-2)]:
    celdas = {}
    # Las celdas de esta fila
    tds = tr.cssselect('td')
    celdas['fecha'] = tds[0].text_content()
    celdas['hora'] = tds[1].text_content()
    # Puede que los valores todavia no esten ... los ignoro
    # Para ello veo que la celda tiene el caracter "-" que no es convertible al tipo float.
    try:
      # Precipitacion en Barra do Facao
      celdas['pBF'] = float( tds[2].text_content() )
      # Precipitacion en Passo Fundo
      celdas['pPF'] = float( tds[3].text_content() )
      data.append(celdas)
    except:
      loggin.debug("\tSe encontraron valores de precipitacion sin datos (-), que no fueron cargados")  
  return data

def main():
  # Configuracion de logging
  logging.basicConfig(level=LOGLEVELS['debug'], format="%(levelname)-8s %(message)s", filename=LOG_FILENAME )
  logging.debug('[%s] - Tratando de conectar a la pagina: %s' % ( datetime.datetime.now() ,LLUVIA_PASSO_FUNDO) )
  dataPF = obtieneDatos(LLUVIA_PASSO_FUNDO)
  logging.debug('[%s] - Tratando de guardar datos en BD: %s' % ( datetime.datetime.now() ,DATABASE) )
  guardaLluviaPF (dataPF)
  logging.debug('[%s] - Fin')

if __name__ == "__main__":
  main()
