# การดึงค่าของการตั้งค่าหลักมาใช้


ส่วนการตั้งค่าหลักๆ ของเว็บเราจะอยู่ที่โฟลเดอร์ `app/config` ในบทนี้เราจะมาดูว่า laravel เตรียมฟังก์ชันอะไร ให้เราใช้ในการดึงค่าจากไฟล์ทั้งหลายในโฟลเดอร์ config ออกมาใช้ได้บ้าง.

laravel เตรียม class ที่ชื่อว่า `Config` ไว้ให้เราเเล้วนะครับ

**ยกตัวอย่างการดึงค่า timezone ออกมา**

	Config::get('app.timezone');

เราสามารถกำหนดค่าของตัวแปรนั้นใหม่ได้ กรณีที่รูปแบบไม่เป็นไปตามที่เราต้องการ:

	$timezone = Config::get('app.timezone', 'UTC');

สังเกตุว่าถ้าเป็นการเข้าถึงค่าในอาเรย์ของไฟล์ laravel จะใช้เครื่องหมายดอท ในการเข้าถึงนะครับ

**กำหนดค่าแบบไม่ต้องเข้าไปในไฟล์เลย**

	Config::set('database.default', 'sqlite');

การกำหนดค่าแบบนี้จะไม่ไปเขียนทับการตั้งค่าในไฟล์ app.php นะครับ จะเกิดผลเฉพาะตรงที่เราประกาศไว้เท่านั้น.

<a name="environment-configuration"></a>
## การกำหนดชุดรูปแบบของการตั้งค่าพื้นฐาน

ในการพัฒนาเว็บเรามักจะเปิด การตั้งค่าต่างๆเพื่อที่จะเอื้ออำนวยให้เราทราบข้อมูล ได้มากที่สุด
แต่ในกรณีที่เว็บออนไลน์แล้วการแสดง การแสดงข้อมูลการทำงานผิดพลาด การลืมไปแล้วว่าเคยทิ้งคำสั่ง debug ไว้ตรงไหน 

เริ่มต้นสร้างไฟล์ชุดการตั้งค่าในโฟลเดอร์ `config` ยกตัวอย่างชื่อ `local`.ยกตัวอย่างการตั้งค่าในไฟล์ สมมุติเราต้องการใช้แคชแบบ file ก็ทำแบบตัวอย่างเลยครับ

	<?php

	return array(

		'driver' => 'file',

	);

> **Note:**  testing เป็นชื่อที่ถูกกำหนด ไว้กับ laravel แล้วว่าถ้าอยู่ในชื่อนี้การตั้งค่าทั้งหมดจะอยู่ในการโหมด unit test ฉะนั้น
เราอย่าไปตั้งทับมันเลยครับ

ส่วนการตั้งค่าที่เราไม่ได้ตั้งไว้ จะอ้างอิงกลับไปที่ไฟล์หลักนะครับ 

ต่อมาเราต้องไปตั้งค่าให้ตัว laravel รู้ว่าขณะนี้อยู่ในโหมดไหน โดยเข้าไปตั้งค่าที่ `bootstrap/start.php` ตัวโฟลเดอร์จะอยู่ข้างหน้าสุดเลย. เข้าไปค้นหา `$app->detectEnvironment` ตัวฟังชันจะใช้ค้นหารูปแบบการตั้งค่าของเว็บเรา 

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

เราก็จะเปลี่ยนให้้เป็นเหมือนตัวอย่าง

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});


**ตัวอย่างการเรียกใช้**

	$environment = App::environment();

<a name="maintenance-mode"></a>
## การปรับปรุงเว็บไซต์

เมื่อเราต้องการปิดเว็บเพื่อทำการปรับปรุง เราจะกำหนดเมทอด `App::down` ไว้ที่ `app/start/global.php` ซึ่งจะทำให้ทุกคำร้องถูกพาไปที่หน้า ที่บอกว่าตอนนี้ เว้บกำลังอยู่ในสถานะปรับปรุง.

ต้องการทำให้รวดเร็วขึ้นก็ใช้ command line ก็ได้

	php artisan down

 up เป็นคำสั่งให้เว็บกลับไปอยู่ในสถานะออนไลน์อีกครั้ง

	php artisan up
ถ้าต้องการเปลี่ยนหน้าที่ใช้ในการแสดงผลก็เข้าไปตั้งค่าที่
`app/start/global.php` ตัวอย่าง

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});