==============================
I3C Master Initialization Flow
==============================

.. kernel-render:: DOT
   :alt: I3C Master Initialization digraph
   :caption: I3C Master Initialization Flow

   digraph master_init {
       ranksep=.25; size="35,35";
       subgraph cluster_0 {
           style=dashed
           x0[shape=ellipse, label="I3C driver probe", style="filled"]
           x1[shape=diamond, label="Secondary Master ?"]
           a1[shape=box, label="Do I3C master controller specific initialization"]
           b1[shape=box, label="Do I3C master controller specific initialization,\n
           except enabling the DEFSLVS interrupt."]
           a2[shape=box, label="Call i3c_primary_master_register"]
           b2[shape=box, label="Call i3c_secondary_master_register"]
           x0->x1;
           x1->a1[label="No"];
           x1->b1[label="Yes"];
           a1->a2; b1->b2;
       }

       subgraph cluster_1 {
           style=dashed
           label="Current Master";
           a3[shape=ellipse, label="i3c_primary_master_register", style="filled"]
           a4[shape=box, label="Initialize I3C master controller object."]
           a5[shape=box, label="Create I2C objects for devices in DTS and add to I2C list."]
           a6[shape=box, label="Set appropriate bus mode based on I2C devices information."]
           a7[shape=box, label="Create a work queue."]
           a8[shape=box, label="Call i3c_primary_master_bus_init"]
           a19[shape=box, label="Add I3C object representing this master to the system."]
           a20[shape=box, label="Expose our I3C bus as an I2C adapter so that I2C devices\n
           are exposed through the I2C subsystem."]
           a21[shape=box, label="Register all I3C devices."]
           a22[shape=box, label="Broadcast ENEC MR, HJ message."]
           a3->a4->a5->a6->a7->a8; a19->a20->a21->a22;
           a8->a19[style="invisible"];
       }

       subgraph cluster_2 {
           style=dashed
           label="Current Master";
           a9[shape=ellipse, label="i3c_primary_master_bus_init", style="filled"]
           a10[shape=box, label="Call bus_init to do controller specific bus initialization\n
           and enabling the controller."]
           a11[shape=box, label="Create I3C object representing the master and add it to\n
           the I3C device list."]
           a12[shape=box, label="Set current master to the object created to represent I3C\n
           master device."]
           a13[shape=box, label="Reset all dynamic address that may have been assigned before."]
           a14[shape=box, label="Disable all slave events before starting DAA."]
           a15[shape=box, label="Pre-assign dynamic address and retrieve device information."]
           a16[shape=box, label="Do dynamic address assignment to all I3C devices currenly\n
           present on the bus."]
           a17[shape=box, label="Create I3C objects representing devices found during DAA."]
           a18[shape=box, label="Send DEFSVLS message containing information about all\n
           known I3C and I2C devices."]
           a9->a10->a11->a12->a13->a14->a15->a16->a17->a18;
       }

       subgraph cluster_3 {
           style=dashed
           label="Current Master";
           h1[shape=ellipse, label="MR request interrupt", style="filled"]
           h2[shape=box, label="Bottom half i3c_master_yield_bus"]
           h1->h2;
       }

       subgraph cluster_4 {
           style=dashed
           label="Current Master";
           i1[shape=ellipse, label="i3c_master_yield_bus", style="filled"]
           i2[shape=box, label="Check if this device is still a master to handle a case of\n
           multiple MR requests from different devices at a same time."]
           i3[shape=box, label="Broadcast DISEC MR, HJ message.\n
           New master should broadcast ENEC MR, HJ once its usage of bus is done."]
           i4[shape=box, label="Get accept mastership acknowldege from requesting master."]
           i5[shape=box, label="In case of failure reenable MR requests by broadcasting ENEC MR, HJ."]
           i6[shape=box, label="Mastership hand over is done."]
           i1->i2->i3->i4->i5->i6;
       }

       subgraph cluster_5 {
           style=dashed
           b3[shape=ellipse, label="i3c_secondary_master_register", style="filled"]
           b4[shape=box, label="Initialize I3C master controller object."]
           b5[shape=box, label="Create I2C objects for devices in DTS and add to I2C list."]
           b6[shape=box, label="Set appropriate bus mode based on I2C devices information."]
           b7[shape=box, label="Create a work queue."]
           b8[shape=box, label="Call i3c_secondary_master_bus_init."]
           b12[shape=box, label="Allocate memory for defslvs_data, that will be used to pass I3C\n
           device list received in DEFSLVS to the I3C core DEFSLVS processing."]
           b13[shape=box, label="Add I3C object representing this master to the system."]
           b14[shape=box, label="Expose our I3C bus as an I2C adapter so that I2C devices are\n
           exposed through the I2C subsystem."]
           b3->b4->b5->b6->b7->b8; b12->b13->b14;
           b8->b12[style="invisible"]
       }

       subgraph cluster_6 {
           style=dashed
           b9[shape=ellipse, label="i3c_secondary_master_bus_init", style="filled"]
           b10[shape=box, label="Call bus_init to do controller specific bus initialization\n
           and enabling the controller."]
           b11[shape=box, label="Create I3C object representing the master and add it to\n
           the I3C device list."]
           b9->b10->b11;
       }

       subgraph cluster_7 {
           style=dashed
           d1[shape=ellipse, label="DEFSLVS interrupt", style="filled"]
           d2[shape=box, label="Controller driver can chose how to handle I2C devices received\n
           in DEFSLVS."]
           d3[shape=box, label="Read all I3C devices information from DEFSLVS message\n
           to defslvs_data structure in the master object."]
           d4[shape=box, label="Call i3c_master_process_defslvs."]
           d1->d2->d3->d4;
       }

       subgraph cluster_8 {
           style=dashed
           d5[shape=ellipse, label="i3c_master_process_defslvs", style="filled"]
           d6[shape=box, label="Acquire I3C bus mastership."]
           d7[shape=diamond, label="Bus acquired ?"]
           d8[shape=box, label="Handle I3C device list from DEFSLVS."]
           d9[shape=box, label="Call i3c_master_populate_bus"]
           d10[shape=box, label="Queue defslvs processing task for retry."]
           d5->d6;
           d7->d8[label="Yes"];
           d8->d9;
           d7->d10[label="No"];
           d6->d7[style="invisible"];
           d10->d5[style="dotted", constraint=false];
       }

       subgraph cluster_9 {
           style=dashed
           e1[shape=ellipse, label="i3c_master_acquire_bus", style="filled"]
           e2[shape=box, label="If device is already holding the mastership,\n
           just broadcast DISEC MR, HJ message and return success."]
           e3[shape=box, label="Call request_mastership method"]
           e1->e2->e3;
       }

       subgraph cluster_10 {
           style=dashed
           f1[shape=ellipse, label="request_mastership", style="filled"]
           f2[shape=box, label="Return success if device is already in master mode."]
           f3[shape=box, label="Return error if device don't have dyn_addr."]
           f4[shape=box, label="Return error if MR requests are disabled by current master."]
           f5[shape=box, label="Send MR request"]
           f6[shape=box, label="Wait with timeout until MR_DONE interrupt is received."]
           f1->f2->f3->f4->f5->f6;
       }

       subgraph cluster_11 {
           style=dashed
           g1[shape=ellipse, label="i3c_master_populate_bus", style="filled"]
           g2[shape=box, label="Free up all I3C addresses to handle address\n
           re assignment by main master."]
           g3[shape=box, label="Master acquire dyn_addr received from the driver."]
           g4[shape=box, label="For every I3C device in the defslvs_data\n
           call i3c_master_add_i3c_dev_locked."]
           g1->g2->g3->g4;
       }

       a2->a3; a8->a9; a18->a19;
       b2->b3; b8->b9; b11->b12;
       b14->d1[style=invisible];
       a22->h1[style=invisible];
       d4->d5;
       d6->e1;
       e3->f1;
       f6->d7[constraint=false];
       h2->i1;
       d9->g1;
       a18->d1[style="dotted", constraint=false];
       f5->h1[style="dotted", constraint=false];
       i4->f6[style="dotted", constraint=false];
   }
