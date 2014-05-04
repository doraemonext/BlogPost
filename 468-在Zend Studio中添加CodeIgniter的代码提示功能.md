**转自：[http://tanteng.sinaapp.com/2013/01/zend-studio-codeigniter](http://tanteng.sinaapp.com/2013/01/zend-studio-codeigniter)**

如何在Zend Studio中添加CodeIgniter框架的代码提示功能呢？

把以下代码分别增加到 `CI_Controller` 和 `CI_Model` 的类声明中，也就是注释里面：

	/*
	 * @property CI_Loader $load
	 * @property CI_DB_active_record $db
	 * @property CI_Calendar $calendar
	 * @property Email $email
	 * @property CI_Encrypt $encrypt
	 * @property CI_Ftp $ftp
	 * @property CI_Hooks $hooks
	 * @property CI_Image_lib $image_lib
	 * @property CI_Language $language
	 * @property CI_Log $log
	 * @property CI_Output $output
	 * @property CI_Pagination $pagination
	 * @property CI_Parser $parser
	 * @property CI_Session $session
	 * @property CI_Sha1 $sha1
	 * @property CI_Table $table
	 * @property CI_Trackback $trackback
	 * @property CI_Unit_test $unit
	 * @property CI_Upload $upload
	 * @property CI_URI $uri
	 * @property CI_User_agent $agent
	 * @property CI_Validation $validation
	 * @property CI_Xmlrpc $xmlrpc
	 * @property CI_Zip $zip
	 */
