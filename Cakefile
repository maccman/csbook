fs      = require('fs')
fp      = require('path')
coffee  = require('coffee-script')
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
  
  title = doc.find('title:first').remove().text()
  
  rootSection = doc.find('> section:first')
  rootSection.replaceWith(rootSection.html())
    
  doc.find('> section').setTag('sect1')
  doc.find('section').setTag('sect2')
  doc.find('section').setTag('sect3')

  doc.find('script').remove()

  doc.find('imagedata').each ->
    element = $(@)
    element.attr('fileref', 
      el.attr('fileref').replace(
        /^\/images\//, 'figs/'
      )
    )

  doc.find('screen').each ->
    element = $(@)
    
    if (element.text().indexOf('//= CoffeeScript') isnt -1)
      element.text(
        element.text().replace(
          /\/\/= CoffeeScript.?\n?/, ''
        )
      )
      
      try
        element.after($('<screen />').text(coffee.compile(element.text())))
        element.after($('<para />').text('Compiles down to:'))
      catch e
        console.error(e)
        console.log(element.text())
  
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