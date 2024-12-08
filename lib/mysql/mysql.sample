/*
MySQL/MariaDB连接器, 单例模式
*/

package mysql

import (
	"fmt"
	"sync"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"gorm.io/gorm/schema"
	db_config "robo.vin/config/mysql"
)

var (
	dbInstance = make(map[string]*gorm.DB)
	mutex      sync.Mutex
)

type Database struct {
	db *gorm.DB
}

func Connect(dbName ...string) (*Database, error) {
	// 加锁确保并发安全
	mutex.Lock()
	defer mutex.Unlock()

	// 如未指定数据库名, 则使用默认数据库
	dbn := "default"
	if len(dbName) > 0 {
		dbn = dbName[0]
	}

	// 如已存在连接实例, 则直接返回
	if db, ok := dbInstance[dbn]; ok {
		dbIns, err := db.DB()
		if err != nil || dbIns.Ping() != nil {
			delete(dbInstance, dbn)
		}
		return &Database{db: db}, nil
	}

	// 获取数据库配置
	dbConfig, ok := db_config.Db[dbn]
	if !ok {
		return nil, fmt.Errorf("database config for %s not found", dbn)
	}

	// 验证数据库配置
	if err := validateDatabaseConfig(dbConfig); err != nil {
		return nil, fmt.Errorf("database config for %s is invalid: %w", dbn, err)
	}

	// 构建数据源串
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s&parseTime=True&loc=Local",
		dbConfig.User,
		dbConfig.Password,
		dbConfig.Host,
		dbConfig.Port,
		dbConfig.Database,
		dbConfig.Charset)

	gormConfig := &gorm.Config{
		Logger: getLogger(dbConfig.Debug), // 调试模式
		NamingStrategy: schema.NamingStrategy{
			SingularTable: true, // 模型映射表名不自动转复数
			NoLowerCase:   true, // 模型映射表明字段不自动转小写
		},
	}

	// 连接数据库
	db, err := gorm.Open(mysql.Open(dsn), gormConfig)
	if err != nil {
		return nil, fmt.Errorf("failed to connect to database %s: %w", dbn, err)
	}

	// 新创建的连接实例存储到dbInstance中
	dbInstance[dbn] = db

	// 赋值并返回数据库实例
	return &Database{db: db}, nil
}

// Database结构体方法, 获取Gorm数据库实例
func (db *Database) GetDB() *gorm.DB {
	return db.db
}

// Exec执行非查询sql语句
func (db *Database) Exec(sql string, args ...interface{}) error {
	return db.db.Exec(sql, args...).Error
}

// Query执行查询sql语句
func (db *Database) Query(dest interface{}, sql string, args ...interface{}) error {
	err := db.db.Raw(sql, args...).Scan(dest).Error
	if err != nil && err != gorm.ErrRecordNotFound {
		return err
	}

	return nil
}

// 返回Gorm日志器
func getLogger(debug bool) logger.Interface {
	if debug {
		return logger.Default.LogMode(logger.Info)
	} else {
		return logger.Default.LogMode(logger.Silent)
	}
}

// 对数据库进行连接
func validateDatabaseConfig(dbConfig config.DBConfig) error {
	if dbConfig.Host == "" {
		return fmt.Errorf("db host is not configured")
	}

	if dbConfig.Port == 0 {
		return fmt.Errorf("db port is not configured")
	}

	if dbConfig.User == "" {
		return fmt.Errorf("db user is not configured")
	}

	if dbConfig.Password == "" {
		return fmt.Errorf("db password is not configured")
	}

	if dbConfig.Database == "" {
		return fmt.Errorf("db database is not configured")
	}

	if dbConfig.Charset == "" {
		return fmt.Errorf("db charset is not configured")
	}

	return nil
}
