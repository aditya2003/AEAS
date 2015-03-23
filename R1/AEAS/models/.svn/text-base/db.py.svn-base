# -*- coding: utf-8 -*-

#########################################################################
## This scaffolding model makes your app work on Google App Engine too
## File is released under public domain and you can use without limitations
#########################################################################

## if SSL/HTTPS is properly configured and you want all HTTP requests to
## be redirected to HTTPS, uncomment the line below:
# request.requires_https()

if not request.env.web2py_runtime_gae:
    ## if NOT running on Google App Engine use SQLite or other DB
    db = DAL('sqlite://storage.sqlite',pool_size=1,check_reserved=['all'])
else:
    ## connect to Google BigTable (optional 'google:datastore://namespace')
    db = DAL('google:datastore')
    ## store sessions and tickets there
    session.connect(request, response, db=db)
    ## or store session in Memcache, Redis, etc.
    ## from gluon.contrib.memdb import MEMDB
    ## from google.appengine.api.memcache import Client
    ## session.connect(request, response, db = MEMDB(Client()))

## by default give a view/generic.extension to all actions from localhost
## none otherwise. a pattern can be 'controller/function.extension'
response.generic_patterns = ['*'] if request.is_local else []
## (optional) optimize handling of static files
# response.optimize_css = 'concat,minify,inline'
# response.optimize_js = 'concat,minify,inline'

#########################################################################
## Here is sample code if you need for
## - email capabilities
## - authentication (registration, login, logout, ... )
## - authorization (role based authorization)
## - services (xml, csv, json, xmlrpc, jsonrpc, amf, rss)
## - old style crud actions
## (more options discussed in gluon/tools.py)
#########################################################################

from gluon.tools import Auth, Crud, Service, PluginManager, prettydate
auth = Auth(db)
crud, service, plugins = Crud(db), Service(), PluginManager()

## create all tables needed by auth if not custom tables
auth.define_tables(username=False, signature=False)

## configure email
mail = auth.settings.mailer
mail.settings.server = 'logging' or 'smtp.gmail.com:587'
mail.settings.sender = 'you@gmail.com'
mail.settings.login = 'username:password'

## configure auth policy
auth.settings.registration_requires_verification = False
auth.settings.registration_requires_approval = False
auth.settings.reset_password_requires_verification = True

## if you need to use OpenID, Facebook, MySpace, Twitter, Linkedin, etc.
## register with janrain.com, write your domain:api_key in private/janrain.key
from gluon.contrib.login_methods.rpx_account import use_janrain
use_janrain(auth, filename='private/janrain.key')

#########################################################################
## Define your tables below (or better in another model file) for example
##
## >>> db.define_table('mytable',Field('myfield','string'))
##
## Fields can be 'string','text','password','integer','double','boolean'
##       'date','time','datetime','blob','upload', 'reference TABLENAME'
## There is an implicit 'id integer autoincrement' field
## Consult manual for more options, validators, etc.
##
## More API examples for controllers:
##
## >>> db.mytable.insert(myfield='value')
## >>> rows=db(db.mytable.myfield=='value').select(db.mytable.ALL)
## >>> for row in rows: print row.id, row.myfield
#########################################################################

## after defining tables, uncomment below to enable auditing
# auth.enable_record_versioning(db)

db.define_table('staff',
  db.Field('name','string'), 
  db.Field('email','string',unique=True,required=True,notnull=True,requires=IS_EMAIL()))
  
  
db.define_table('branches',
  db.Field('code','string',length=2,unique=True,notnull=True),
  db.Field('name','string',requires=IS_IN_SET(('EHD','CND','CSE','ECE','ECD','CLD','CSD'))),
  db.Field('program_coordinator',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s'),label='staff name'))



db.define_table('student',
  db.Field('rollno','integer'),
  db.Field('name','string'), 
  db.Field('email','string',unique=True,required=True,notnull=True,requires=IS_EMAIL()),
  db.Field('parent_email','string',unique=True,required=True,notnull=True,requires=IS_EMAIL()),
  db.Field('branch',db.branches,requires=IS_IN_DB(db,'branches.code','%(name)s'),label='branch name'))
  
db.define_table('courses', 
 db.Field('courseid','string',notnull=True,required=True,unique=True,label='Course Id '),
 db.Field('coursename','string',notnull=True,unique=True,required=True,label='Course Name'), #unique=True also creates index for speed
 db.Field('credits','integer',required=True,notnull=True,requires=IS_IN_SET(('2','3','4','5')))) 
 
  

db.define_table('current_sem',
      db.Field('course_id',db.courses,requires=IS_IN_DB(db,'courses.courseid','%(coursename)s'),label='course name'), 
      db.Field('instructor','list:reference db.staff',required=True,notnull=True,label='Instructor'), #Optional error in syntax
      db.Field('sem_year','integer'), 
      db.Field('sem_season','string'))



db.define_table('grade',
                       db.Field('grade_name',requires=IS_IN_SET(('A','A-','B','B-','C','C-','D','F'))),
                       db.Field('grade_point','integer',requires=IS_IN_SET(('10','9','8','7','6','5','4','2'))))
 
  
   
db.define_table('status',
 db.Field('coursename',label='Course name'), 
 db.Field('sem_year','integer'), 
 db.Field('sem_season','string'),
 db.Field('rollno',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
 db.Field('acqgrade',db.grade,requires=IS_IN_DB(db,'grade.point','%(name)s'),label='Acquired Grade'))


#sem value default set

db.define_table('warning',
    db.Field('subject','string',required=True,notnull=True),
    db.Field('issuedto',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
   db.Field('issuedby',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s')),
    db.Field('in_sem_year',db.current_sem,requires=IS_IN_DB(db,'sem_year')),
    db.Field('in_sem_season',db.current_sem,requires=IS_IN_DB(db,'sem_season')),
    db.Field('course',db.current_sem,requires=IS_IN_DB(db,'courses.id','%(coursename)s')),
    db.Field('dean',requires=IS_IN_SET(('y','n'))),
    db.Field('parent',requires=IS_IN_SET(('y','n'))),
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))
    
db.define_table('message_student',
    db.Field('subject','string',required=True,notnull=True),
    db.Field('rollno',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
    db.Field('tofrom',requires=IS_IN_SET(('to','from'))),  #to indicates - from PC to student
    db.Field('in_sem_year',db.current_sem,requires=IS_IN_DB(db,'sem_year')),
    db.Field('in_sem_season',db.current_sem,requires=IS_IN_DB(db,'sem_season')),
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))
    
db.define_table('message_staff',
    db.Field('subject','string',required=True,notnull=True),
    db.Field('from',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s')),
    db.Field('to',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s')),
    db.Field('in_sem_year',db.current_sem,requires=IS_IN_DB(db,'sem_year')),
    db.Field('in_sem_season',db.current_sem,requires=IS_IN_DB(db,'sem_season')),
    db.Field('inconcern',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))


db.define_table('message_staff_thread',
    db.Field('message_id',db.message_staff,requires=IS_IN_DB(db,'message_staff.id','%(subject)s')),
    db.Field('from',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s'),label='staff name'),
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))
    
    
db.define_table('message_student_thread',
    db.Field('message_id',db.message_staff,requires=IS_IN_DB(db,'message_staff.id','%(subject)s')),
    db.Field('rollno',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
    db.Field('tofrom',requires=IS_IN_SET(('to','from'))),     
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))    
    
    
db.define_table('warning_thread',
    db.Field('warning_id',db.warning,requires=IS_IN_DB(db,'warning.id','%(subject)s')),
    db.Field('staff_involved',db.staff,requires=IS_IN_DB(db,'staff.id','%(name)s'),label='staff name'),
    db.Field('student_involved',db.student,requires=IS_IN_DB(db,'student.rollno','%(name)s')),
    db.Field('matter','text'),
    db.Field('timestamp','datetime',writable=False))
