.. link: 
.. description: 
.. tags: code, AppEngine, html
.. date: 2011-04-03 11:29:45
.. slug: file-upload-gotcha
.. title: File upload gotcha...
.. description: In which I note a silly mistake I made while writing a webapp's upload form.
.. comments: true
.. wordpress_id: 38

.. role:: python(code)
   :language: python

I'm currently working on some much-needed updates to an AppEngine service that processes and serves content based on a CSV version of a spreadsheet.  One of these new features is the ability to upload the spreadsheet via an HTML form.  Of course, being early-ish on a Sunday morning, there was a little head scratching...

.. code:: html

    <html>
      <body>
        <form action="/admin" method="post">
          <div><input type="file" name="csvfile" accept="text/csv"></div>
          <div><input type="submit" value="Update CSV"></div>
        </form>
      </body>
    </html>

Looks ok, right? Not quite. :python:`self.request.get("csvfile")` only returns the file name!  I must be forgetting something.  After my coffee kicked in, I took another look at it and facepalmed.  The following modification returned what I wanted.

.. code:: html

    <form action="/admin" enctype="multipart/form-data" method="post">

Sure enough, adding the "enctype" attribute has my code returning the contents of the CSV file.

Moral of the story? Don't code before your morning intake of caffeine.