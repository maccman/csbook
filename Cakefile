fs      = require('fs')
fp      = require('path')
$       = require('jQuery')
{spawn} = require('child_process')
input   = './chapters'
output  = './docbook'

$.fn.setTag = (tag) ->
  $(@).each ->
    element = $("<#{tag}/>")
    element.html($(@).html())
    for attr in @attributes
      element.attr(attr.name, $(@).attr(attr.name))
    $(@).replaceWith(element)

format = (data) ->
  doc = $('<body />').html(data)
  doc.find('section').removeAttr('id')
  doc.find('> section').setTag('sect1')
  doc.find('section').setTag('sect2')
  doc.find('script').remove()
  doc.find('imagedata').each ->
    element = $(@)
    newSrc  = el.attr('fileref').replace(/^\/images\//, 'figs/')
    element.attr('fileref', newSrc)
  
  title = doc.find('title:first').text()
  # doc.find('> sect1:first').replaceWith(doc.find('> sect1:first').html())
  data  = doc.html()
  
  """
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE chapter PUBLIC "-//OASIS//DTD DocBook XML V4.5//EN"
  "http://www.oasis-open.org/docbook/xml/4.5/docbookx.dtd">
  <chapter>
  <title>#{title}</title>
   #{data}
  </chapter>
  """

build = (child) ->
  pandoc = spawn 'pandoc', ['-f', 'markdown', '-t', 'docbook']
  
  result = ''
  
  pandoc.stderr.on 'data', (data) ->
    process.stderr.write data.toString()
    
  pandoc.stdout.on 'data', (data) ->
    result += data.toString()
  
  pandoc.stdout.on 'end', (data) ->
    outputChild = fp.join(output, fp.basename(child, '.md') + '.xml')
    fs.writeFileSync(outputChild, format(result))
  
  md = fs.readFileSync(fp.join(input, child), 'utf-8')
  pandoc.stdin.write(md)
  pandoc.stdin.end()
  
buildAll = ->
  for child in fs.readdirSync(input)
    build(child)

task 'build', 'Build all', ->
  buildAll()