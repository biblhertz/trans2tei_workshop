# Customization

Our document conversion code was developed on real-life documents and thus is limited to what we encountered there.
At the same time, we left some gaps open for your and our own extensions.

1. Easiest is to copy `simplify.py` to a different name to start with a working example
2. Take out the steps you do not need
3. For your own additional XSLT conversions, you can add them in the same manner as present in `transform.py` or – as 
   a shortcut – use the wrapper function `base()` directly in your workflow script.
4. For your replacements, feel free to add to `replacements.py` or your own copy of this file.
5. All the print statement are non-essential and only serve for debugging.
