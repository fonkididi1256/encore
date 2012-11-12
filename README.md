encore
======

my project
Index: contrib/legal/legal.install
===================================================================
--- contrib/legal/legal.install
+++ contrib/legal/legal.install
@@ -118,5 +118,6 @@
 function legal_uninstall() {
   drupal_uninstall_schema('legal');
   variable_del('legal_display');
+  variable_del('legal_new_window');
 }
 
Index: contrib/legal/legal.module
===================================================================
--- contrib/legal/legal.module
+++ contrib/legal/legal.module
@@ -115,7 +115,11 @@
             $form['legal']['conditions'] = array(
               '#value' => ' ',
             );
-  					$accept_label = t('<strong>Accept</strong> !terms of Use', array('!terms' => l('Terms & Conditions', 'legal')));
+            if (variable_get('legal_new_window', '0') == '1') {
+              $accept_label = t('<strong>Accept</strong> !terms of Use', array('!terms' => l('Terms & Conditions', 'legal', array('attributes' => array('target' => '_blank')))));
+            } else {
+              $accept_label = t('<strong>Accept</strong> !terms of Use', array('!terms' => l('Terms & Conditions', 'legal')));
+            }
             break;  
 
         default: // scroll box (HTML)
@@ -186,6 +190,7 @@
       '#default_value' => $conditions['conditions'],
       '#description' => t('Your Terms & Conditions'),
       '#required' => TRUE,
+      '#weight' => 4,
     ); 
     
     // overide accept checbox requirement on preview 
@@ -199,8 +204,19 @@
       '#options' => array(t('Scroll Box'), t('Scroll Box (CSS)'), t('HTML Text'), t('Page Link')),
       '#description' => t('How terms & conditions should be displayed to users.'),
       '#required' => TRUE,
+      '#weight' => 0,
     );
     
+    if (variable_get('legal_display', '0') == 3) {
+      $form['new_window'] = array(
+        '#type' => 'checkbox',
+        '#title' => t('Open Page Link in new window?'),
+        '#default_value' => variable_get('legal_new_window', '0'),
+        '#weight' => 1,
+      );
+    }
+    
+    
     // additional checkboxes
     $form['extras'] = array(
       '#type' => 'fieldset',
@@ -209,6 +225,7 @@
       '#collapsible' => TRUE,
       '#collapsed' => TRUE,
       '#tree' => TRUE,
+      '#weight' => 5,
     );
   
     $extras_count = count($conditions['extras']);
@@ -241,6 +258,7 @@
       '#description' => t('Explain what changes were made to the T&C since the last version. This will only be shown to users who accepted a previous version. Each line will automatically be shown as a bullet point.'),
       '#collapsible' => TRUE,
       '#collapsed' => TRUE,
+      '#weight' => 6,
     );
     
     $form['changes']['changes'] = array(
@@ -253,11 +271,13 @@
     $form['preview'] = array(
       '#type' => 'button',
       '#value' => t('Preview'),
+      '#weight' => 3,
     );
     
     $form['save'] = array(
       '#type' => 'submit',
       '#value' => t('Save'),
+      '#weight' => 3,
     );
     
     return $form;
@@ -318,8 +338,13 @@
 
 function legal_administration_submit($form, &$form_state) {
 
-	$values = $form_state['values'];    
-	if ($form_state['clicked_button']['#value'] == t('Preview')) return;
+  $values = $form_state['values'];
+  
+  if (isset($form_state['values']['new_window'])) {
+    variable_set('legal_new_window', $form_state['values']['new_window']);
+  }
+  
+  if ($form_state['clicked_button']['#value'] == t('Preview')) return;
 
   if ( variable_get('legal_display', '0') !=  $form_state['values']['display'] ) {
       variable_set('legal_display', $form_state['values']['display']);