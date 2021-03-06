# GOLANG interface for OpenStreetMap API v0.6

### Introduction

Please see more inforamtion in OSM wiki: http://wiki.openstreetmap.org/wiki/API_v0.6

### Installing 

	go get github.com/iostrovok/go-osm-api/osmapi

### How use example.

See example for more inforamtion. They are in "./osmapi/example/" path.

	> mkdir -p /mypath/go
	> cd /mypath/go
	> export OSM_URL="http://api06.dev.openstreetmap.org"
	> export OSM_USER="logit_for_dev_osm_api"
	> export OSM_PASSWD="password_for_dev_osm_api"
	> export GOPATH="/mypath/go"; 
	> go get github.com/iostrovok/go-osm-api/osmapi

	# Creates new node
	> go run ./src/github.com/iostrovok/go-osm-api/osmapi/example/create_node.go -lat=-0.023642 -lon=51.506358

	# Upadte node with id => 12313
	# Sets "name:ru" to "Поселок где-то в ЛОНДОНЕ"
	> go run ./src/github.com/iostrovok/go-osm-api/osmapi/example/modify_node.go -nodeid=12313

	# Delete old point 
	> go run ./src/github.com/iostrovok/go-osm-api/osmapi/example/delete_node.go -nodeid=12313


# Using.

## Nodes

### Common action

	/* Request object creating */
	ChSet, err := req.Changesets("modify") // "delete", "create"
	if err != nil {
		log.Fatal(err)
	}

### Get date of exists way or node (point)

	import "github.com/iostrovok/go-osm-api/osmapi"
	import "log"

	req := osmapi.MyRequest(login, pass)
	if req == nil {
		log.Fatal("Request create")
	}
	
	/* 
		Load node's date for view information.
	*/
	if node, err := req.LoadNodeDate("1442930428"); err != nil {
		log.Fatal("error LoadNodeDate")
	} else {
		/*  Read tags from nodes */
		log.Println( "name:en: "+ node.Tag("name:en"))
		log.Println( "name:ru: "+ node.Tag("name:ru"))
		log.Println( "name:uk: "+ node.Tag("name:uk"))
	}

### Create new node (point)

	import "github.com/iostrovok/go-osm-api/osmapi"
	import "log"

	/* London is calling */
	Lat := -0.023642
	Lon := 51.506358

	req := osmapi.MyRequest(login, pass)
	if req == nil {
		log.Fatal("Request create")
	}
	
	/* Make object for request */
	ChSet, err := req.Changesets("create")
	if err != nil {
		log.Fatal(err)
	}

	ChSet.Generator("My Super System")

	/* Create new node */
	node, err_n := ChSet.NewNode(Lat, Lon)
	if err_n != nil {
		log.Fatal(err_n)
	}

	/*  Set new data */
	node.AddTag("name:en", "Anywhere in London")
	node.AddTag("name:ru", "Поселок где-то в лондоне")
	node.AddTag("name:uk", "Поселок где-то в лондоне")
	node.AddTag("place", "street")

	/* Delete existing tag */
	node.DelTag("name")

	/* Upload new data */
	if newId, err := ChSet.Upload(); err != nil {
		log.Fatal(err)
	} else {
		log.Printf("Create node. Id; %s\n", newId)
	}
	/*
		If you want to see uploaded data without sending you can use do so:
		req.SetDebug(); // eq req.SetDebug(true) 
		if newId, err := ChSet.FakeUpload(); err != nil { 
			log.Fatal(err)
		} else {
			log.Printf("Create node. Id; %s\n", newId)
		}
		req.SetDebug(false); // clean debug 
	*/
	/* Fixed result into OSM */
	if err := ChSet.Close(); err != nil {
		log.Fatal(err)
	}
	/* ChSet is nil now */


### Modify existing node (point)

	import "github.com/iostrovok/go-osm-api/osmapi"
	import "log"

	req := osmapi.MyRequest(login, pass)
	if req == nil {
		log.Fatal("Request create")
	}

	/* Make object for request */
	ChSet, err := req.Changesets("modify")
	if err != nil {
		log.Fatal(err)
	}


	/* 
		FIRST. Load node's date for modification
		"1442930428" is real point
	*/
	if node, err_n := ChSet.LoadNode("1442930428"); err != nil {
		log.Fatal("Node сreate")
	} else {
		/*  Set new tag into data */
		node.AddTag("name:en", "Akultsevo")
		node.AddTag("name:uk", "Акульцево")
		/* Delete existing tag */
		node.DelTag("name")
	}

	/* 
		SECOND. Load node's date for modification
		"1442930461" is real point near "1442930428" :)
	*/
	if node, err := ChSet.LoadNode( "1442930461"); err != nil {
		log.Fatal("Node сreate")
	} else {
		/*  Set new tag into data */
		node.AddTag("name:en", "Petrushikha")
	}
	/* Upload new data */
	if _, err := ChSet.Upload(); err != nil {
		log.Fatal(err)
	}

	/* Fixed result into OSM */
	if err := ChSet.Close(); err != nil {
		log.Fatal(err)
	}
	/* ChSet is nil now */


### Delete existing node (point)

	import "github.com/iostrovok/go-osm-api/osmapi"
	import "log"

	NodeId = "221442930428"

	req := osmapi.MyRequest(login, pass)
	if req == nil {
		log.Fatal("Request create")
	}

	/* Make object for request */
	ChSet, err := req.Changesets("delete")
	if err != nil {
		log.Fatal(err)
	}

	/* Load node's date for deleting */
	if node, err_n := ChSet.LoadNode(NodeId); err != nil {
		log.Fatal("Node сreate")
	} 

	/* Upload new data */
	if _, err := ChSet.Upload(); err != nil {
		log.Fatal(err)
	}

	/* Fixed result into OSM */
	if err := ChSet.Close(); err != nil {
		log.Fatal(err)
	}
	/* ChSet is nil now */

## Ways

### Common action

	/* Request object creating */
	ChSet, err := req.Changesets("modify") // "delete", "create"
	if err != nil {
		log.Fatal(err)
	}

### Read existing way  
	/* Loading existing way data */
	way, err := ChSet.WayLoad("52868")
	if err != nil {
		log.Fatal(err)
	}

### Create new way  

	/* Create new way */
	way, err := ChSet.WayNew()
	if err != nil {
		log.Fatal(err)
	}

### Create and add node into end of way 
	node, err := ChSet.NewNode("0.017183, "51.506286")
	if err != nil {
		log.Fatal(err)
	}
	id, err := ChSet.WayAddNode(node)
	if err != nil {
		log.Fatal(err)
	} 

### Create and add node after node which has id = "123456789"
	node, err := ChSet.NewNode("0.017183, "51.506286")
	if err != nil {
		log.Fatal(err)
	}
	id, err := ChSet.WayAddNode(node, "123456789")
	if err != nil {
		log.Fatal(err)
	} 

### Tag's action 
	node.AddTag("name:tr", "Long way")
	node.DelTag("name")


### Create and add node to first place of way
	node, err := ChSet.NewNode("0.017183, "51.506286")
	if err != nil {
		log.Fatal(err)
	}
	id, err := ChSet.WayAddNode(node, "0")
	if err != nil {
		log.Fatal(err)
	} 

### Delete all nodes from way & changeset
	if err := ChSet.WayDelAllNodes(); err != nil {
		log.Fatal(err)
	}

### Delete node with id "123456789" from way & changeset
	if err := ChSet.WayDelNode("123456789"); err != nil {
		log.Fatal(err)
	} 

## Relations

### Common action

	/* Request object creating */
	ChSet, err := req.Changesets("modify") // "delete", "create"
	if err != nil {
		log.Fatal(err)
	}

### Read existing relation  

	/* Loading existing relation data */
	if _, err := ChSet.RelationLoad("12993"); err_n != nil {
	if err != nil {
		log.Fatal(err)
	}

### Create new relation  

	/* Create new relation */
	way, err := ChSet.RelationNew()
	if err != nil {
		log.Fatal(err)
	}

### Add exists node and ways to relation

	if err := ChSet.RelationAddMember("way", "52868", "outer"); err != nil {
		log.Fatal(err)
	} 

	if err := ChSet.RelationAddMember("node", "52868", "outer"); err != nil {
		log.Fatal(err)
	} 


### Delete all members from members 

	if err := ChSet.RelationDelAllMembers(); err != nil {
		log.Fatal(err)
	}

### Delete node/way with id "52868" from members

	if err := ChSet.RelationDelMember("way", "52868"); err != nil {
		log.Fatal(err)
	} 

	if err := ChSet.RelationDelMember("node", "52868"); err != nil {
		log.Fatal(err)
	} 
