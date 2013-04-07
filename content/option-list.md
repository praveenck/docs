Extension Configuration
=======================

This documents the proposal for defining system configuration options for an
extension.

In the old model, to provide a drop-down selection box with values for the
merchant to select from, you had to write a PHP class source model to return the
array. Now, you can define static option lists in system.xml directly.

Configurable Static Option Lists
--------------------------------

Here are a couple of side-by-side comparisons of the old configuration model and
the new. We'll look at an example in which we get a customer to show up in a
group, and set up whether the VAT number appears in the frontend.

### <options\> + <option\> with string value

In the old old model, you configure the source model and the source_model class
(in this example,  Mage_Backend_Model_Config_Source_Yesno).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  In system.xml:

            <field id="auto_group_assign" translate="label comment"
            type="select" sortOrder="10" showInDefault="1" showInWebsite="1"
            showInStore="1">

                <label>Enable Automatic Assignment to Customer Group</label>

                <comment>To show VAT number on frontend, set Show VAT Number on
            Frontend option to Yes.</comment>

                <source_model>Mage_Backend_Model_Config_Source_Yesno</source_model>

            </field>

        2.  Create Yesno.php:

            class Mage_Backend_Model_Config_Source_Yesno implements
            Mage_Core_Model_Option_ArrayInterface

            {

                /**

                 * Options getter

                 *

                 * @return array

                 */

                public function toOptionArray()

                {

                    return array(

                        array('value' => 1,
            'label'=>Mage::helper('Mage_Backend_Helper_Data')->__('Yes')),

                        array('value' => 0,
            'label'=>Mage::helper('Mage_Backend_Helper_Data')->__('No')),

                    );

                }

                /**

                 * Get options in "key-value" format

                 *

                 * @return array

                 */

                public function toArray()

                {

                    return array(

                        0 =>
            Mage::helper('Mage_Backend_Helper_Data')->__('No'),

                        1 =>
            Mage::helper('Mage_Backend_Helper_Data')->__('Yes'),

                    );

                }

            }
            
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the new model, all you have to do is configure options in system.xml.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  In system.xml:

            <field id="auto_group_assign" translate="label comment"
            type="select" sortOrder="10" showInDefault="1" showInWebsite="1"
            showInStore="1">

                <label>Enable Automatic Assignment to Customer Group</label>

                <comment>To show VAT number on frontend, set Show VAT Number on
            Frontend option to Yes.</comment>

                <options>

                    <option label="Yes">1</option>

                    <option label="No">0</option>

                </options>

            </field>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### <options\> + <option\> with constant value - class::constant

In this example, we're setting up tax calculations based on customer location.

Using the old configuration model, it was necessary to configure the source
model and the source_model class (in this example the class is called
Mage_Customer_Model_Config_Source_Address_Type)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  In system.xml:

            <field id="tax_calculation_address_type" translate="label"
            type="select" sortOrder="10" showInDefault="1" showInWebsite="1"
            showInStore="1">

                <label>Tax Calculation Based On</label>

                <source_model>Mage_Customer_Model_Config_Source_Address_Type</source_model>

                <depends>

                    <field id="auto_group_assign">1</field>

                </depends>

            </field>

        2.  Type.php:

            class Mage_Customer_Model_Config_Source_Address_Type implements
            Mage_Core_Model_Option_ArrayInterface

            {

                /**

                 * Retrieve possible customer address types

                 *

                 * @return array

                 */

                public function toOptionArray()

                {

                    return array(

                        Mage_Customer_Model_Address_Abstract::TYPE_BILLING =>
            Mage::helper('Mage_Customer_Helper_Data')->__('Billing Address'),

                        Mage_Customer_Model_Address_Abstract::TYPE_SHIPPING =>
            Mage::helper('Mage_Customer_Helper_Data')->__('Shipping Address')

                    );

                }

            }

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the new model, all that's necessary is to configure options in system.xml.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  system.xml

            <field id="tax_calculation_address_type" translate="label"
            type="select" sortOrder="10" showInDefault="1" showInWebsite="1"
            showInStore="1">

                <label>Tax Calculation Based On</label>

                <options>

                    <option label="Billing
            Address">{{Mage_Customer_Model_Address_Abstract::TYPE_BILLING}}</option>

                    <option label="Shipping
            Address">{{Mage_Customer_Model_Address_Abstract::TYPE_SHIPPING}}</option>

                </options>

                <depends>

                    <field id="auto_group_assign">1</field>

                </depends>

            </field>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configurable Dynamic Option Lists
---------------------------------

### <options\> with service-call attribute

Using the old model, you had to configure both the source_model and the
source_model class (in this example,  Mage_Customer_Model_Config_Source_Group)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  In system.xml:

            <field id="default_group" translate="label" type="select"
            sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="1">

                <label>Default Group</label>

                <source_model>Mage_Customer_Model_Config_Source_Group</source_model>

            </field>

        2.  Group.php:

            class Mage_Customer_Model_Config_Source_Group implements
            Mage_Core_Model_Option_ArrayInterface

            {

                protected $_options;

                public function toOptionArray()

                {

                    if (!$this->_options) {

                        $this->_options =
            Mage::getResourceModel('Mage_Customer_Model_Resource_Group_Collection')

                            ->setRealGroupsFilter()

                            ->loadData()->toOptionArray();

                        array_unshift($this->_options, array('value'=> '',
            'label'=> Mage::helper('Mage_Customer_Helper_Data')->__('-- Please
            Select --')));

                    }

                    return $this->_options;

                }

            }

Using the new model, you only have to configure options for service-call attributes idField and labelField in system.xml.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        1.  In system.xml:

            <field id="default_group" translate="label" type="select"
            sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="1">

                <label>Default Group</label>

                <options service-call="customerGroup"
            idField="customer_group_id" labelField="customer_group_code"/>

            </field>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note: The service specified by the service-call attribute returns a list of objects, each containing data in key-value pairs. In the example above, the customerGroup service returns the following list of objects:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

            groups: {

                object0: {

                        customer_group_id: 1

                        customer_group_code: General

                        tax_class_id: 3

                       }

                object1: {

                        customer_group_id: 2

                        customer_group_code: Wholesale

                        tax_class_id: 3

                       }

                object2: {

                        customer_group_id: 3

                        customer_group_code: Retailer

                        tax_class_id: 3

                       }

                    }

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The idField identifies the option, in this case the customer_group_id, and the labelField identifies the value, in this case the customer_group_code. In the current example, rendering will result in the following html:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

            <option value="1">General</option>

            <option value="2">Wholesale</option>

            <option value="3">Retailer</option>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

