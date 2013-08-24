Angle format specifier for Angle.to_string()
=============================================

This is a proposal to both simplify and enhance flexibility for creating string
representations of ``Angle`` objects.

The current ``Angle.to_string()`` method does a good job of covering a lot of possible use
cases, but this comes at the cost of complexity and special rules (e.g. for ``sep`` the
special string 'fromunit' means 'dms' if the unit is degrees, or 'hms' if the unit is
hours).  Even with this complexity there is limited flexibility to format in unanticipated
ways.

The specific proposal is to define a format mini-language (ala ``strptime``) which is used
to specify a ``format`` keyword arg to ``Angle.to_string``.  This would replace four
args (``decimal``, ``sep``, ``alwayssign``, ``fields``) with one new ``style`` arg, and 
change the definition of the existing ``format`` arg.

The format specifier language could also be used to validate input angle strings.


Format specifier
-----------------

A format specifier is a string which determines how an Angle object is 
formatted.  It can contain arbitrary text or unicode symbols.  Whitespace
is not significant, so the '%sp' directive must be used to generate
a space character.  The following list of directives are defined to allow
flexible formatting::

  %H         Decimal representation in units of hour angle
  %HH        Hour component in units of hour angle
  %HM        Minute component in units of hour angle
  %HS        Second component in units of hour angle (floating point precision
             set by the ``precision`` argument or attribute)

  %D         Decimal representation in units of degrees
  %DD        Degree component in units of degrees
  %DM        Minute component in units of degrees
  %DS        Second component in units of degrees (floating point precision
             set by the ``precision`` argument or attribute)

  %+         Plus sign if the angle is positive, blank otherwise

  %HH_unit   Unit / separator string for an %HH value, defined by ``style``
  %HM_unit   Unit / separator string for an %HM value, defined by ``style``
  %HS_unit   Unit / separator string for an %HS value, defined by ``style``

  %DD_unit   Unit / separator string for a %DD value, defined by ``style``
  %DM_unit   Unit / separator string for a %DM value, defined by ``style``
  %DS_unit   Unit / separator string for a %DS value, defined by ``style``

  %HMS       Equivalent to '%HH %H_hour %HM %H_min %HS %H_sec'
  %DMS       Equivalent to '%DH %D_hour %DM %D_min %DS %D_sec'

  %sp        Space character
  %%         Percent character


Styles
-------

The ``style`` parameter defines the set of unit / separator strings as follows::

  style:    'hms-dms'  'spaces'   'colons'   'unicode'        'latex' 
            ---------  --------   --------   ---------   -------------------
  %HH_unit      'h'       ' '        ':'        ' '       r'^\mathrm{h}'      
  %HM_unit      'm'       ' '        ':'        ' '       r'^\mathrm{m}'       
  %HS_unit      's'       ''         ''         ' '       r'^\mathrm{s}'      
  %DD_unit      'd'       ' '        ':'        '°'       r'^\circ'           
  %DM_unit      'm'       ' '        ':'        " "       r'{}^\prime'        
  %DS_unit      's'       ''         ''         ' '       r{}^{\prime\prime}''


(NOTE: I seem unable to paste the correct unicode characters in the 'unicode' column above).

Examples
--------
::

  # Basic stuff
  a = Angle('10.25d')
  a.to_string(format='%HMS')  # default h m s separators
  a.to_string(format='%DMS', style='colons')  # colon-separated

  # Customizing a bit within predefined structure
  a.to_string(format='%+ %HMS')  # Equivalent to alwayssign=True
  a.to_string(format='%DH %DH_unit %DM %DM_unit', style='latex')
  a.to_string(format='%HMS', precision=0)  # 0 digits precision instead of default=5
  a.to_string(format='%D %DD_unit', style='unicode', precision=3)  # nice unicode degree symbol

  # WHATEVER you want!
  a.to_string(format='%DH : %DM')
  a.to_string(format='%DH - %DM - %DS')
  a.to_string(format=u'%D °', precision=3)  # nice unicode degree symbol, hardcoded
  a.to_string(format='%HH %sp hours, %sp %HM %sp minutes, %sp %HS %sp seconds')


Other candidate ideas
----------------------

Precision
^^^^^^^^^^^

We could support specifying precision expicitly in the format string with a syntax like::

  %H.ddd      # three digits
  %D.         # zero digits (prints without decimal point)
  %D.d        # one digit
  %DS.ddddd   # five decimal digits

An explicitly specified ``precision`` arg would take precedence.  In this case
the format() method would normally have precision=None and then you would define (e.g.)::

  %HMS = '%HH %H_hour %HM %H_min %HS.dddd %H_sec'
  %DMS = '%DH %D_hour %DM %D_min %DS.dddd %D_sec'

It might also be possible to go the other way and have the ``precision`` arg take precedence.

Units
^^^^^^^^^^^

It might be possible to cut the `unit` arg as well by putting it in the format.  E.g.::

  %U[microarsec]   Decimal representation in microarcsec
  %U[radian].ddd   Decimal representation in radians with 3 digits precision
  %U_microarsec    Decimal representation in microarcsec (alternate syntax)
  %unit[radian]    Decimal representation in microarcsec (alt syntax 2)
